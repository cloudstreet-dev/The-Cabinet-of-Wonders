# HyperLogLog

You have a stream of a billion IP addresses with many repeats. You want to know, approximately, how many distinct ones there are. Exact: store every distinct one in a hash set, eight or more bytes per element, gigabytes of RAM. HyperLogLog: store a few thousand bytes total, get the answer to within 2% relative error, regardless of whether the true answer is a hundred or a hundred billion.

The construction is bizarre. You hash each element, look at how many leading zeros there are in the hash's binary representation, keep the maximum. Some additional cleverness with bucketing and harmonic averaging, and that is the whole estimator. The math behind it is several pages of careful asymptotics; the implementation is twenty lines.

## The intuition

Hash an element to a uniform random binary string. Count leading zeros (i.e., bits before the first 1). The probability of seeing \\(k\\) leading zeros is \\(2^{-(k+1)}\\). So observing leading-zero count \\(k\\) suggests you have hashed about \\(2^{k+1}\\) elements.

Specifically: if you have hashed \\(n\\) distinct elements and recorded the maximum leading-zero count seen so far, that maximum is concentrated around \\(\log_2 n\\). You have a one-number compressed representation of the cardinality.

But: the variance of the maximum-of-geometrics is enormous. A single trial with \\(n = 1000\\) might give a max leading-zero count anywhere from 5 to 15. Useless on its own.

## The fix: many independent trials

Run \\(m\\) independent estimates and average them. The catch is that "independent estimate" cannot mean "use a different hash function and re-hash all the elements" — you would touch the data \\(m\\) times.

Instead: hash each element once. Use the first \\(\log_2 m\\) bits of the hash to choose one of \\(m\\) buckets; the remaining bits are the "leading-zero" content for that bucket. Each bucket independently maintains a max-leading-zero counter for the elements that fell into it.

Each bucket sees about \\(n/m\\) elements. Each bucket's counter is a noisy estimate of \\(\log_2(n/m)\\). Average across buckets to reduce variance.

Then: the *harmonic mean* of \\(2^{\text{counter}_i}\\) gives \\(n/m\\), so multiply by \\(m\\) and a small bias correction. The harmonic mean is used because the geometric distribution of leading zeros has heavy outliers, and harmonic mean down-weights large values, giving a tighter estimator.

The full estimator:

\\[ \hat{n} = \alpha_m \cdot m^2 \cdot \left( \sum_{i=1}^{m} 2^{-M_i} \right)^{-1} \\]

where \\(M_i\\) is the maximum leading-zero count in bucket \\(i\\), and \\(\alpha_m\\) is a known constant correcting for bias. The relative error is approximately \\(1.04 / \sqrt{m}\\).

For \\(m = 2^{14} = 16384\\) buckets, error is about 0.8%. Each bucket needs about 6 bits (since the max leading-zero count over 64-bit hashes is at most 64, a 6-bit counter suffices for cardinalities up to \\(2^{64}\\)). Total memory: \\(16384 \times 6 / 8 = 12\\) KB.

For 12 KB you have an unbiased estimator of cardinality with sub-1% error, working from a single pass over the data, using only one hash per element. At the time HyperLogLog was published (2007), this was startling.

## Sketch operations

The sketch is a vector of small counters. Operations on it are simple:

- **Insert(\\(x\\))**: hash \\(x\\), pick bucket from first \\(\log_2 m\\) bits, count leading zeros in remainder, update bucket max.
- **Estimate**: harmonic-mean formula.
- **Merge two sketches**: take the elementwise max of the two bucket vectors. The merged sketch is what you would have computed if you had inserted all elements from both inputs into a single sketch from scratch.

Mergeability is the killer feature. You can shard the data across machines, compute a sketch per shard in parallel, and merge them at the end. The merge is associative and commutative; you can build trees of merges. Cardinality estimation across data centers becomes embarrassingly parallel.

## The dirty corners

The basic estimator is biased for small \\(n\\) (the harmonic mean has a positive bias when many buckets are empty). Practical implementations have a *small-range correction*: when many buckets are still 0, switch to a *linear counting* estimator which is more accurate at small cardinalities. Above a threshold, switch back to HyperLogLog.

There is also a *large-range correction* in the original paper that is no longer needed if you use 64-bit hashes (the original used 32-bit hashes which saturated near \\(2^{32}\\)).

Google extended the algorithm in 2013 (HyperLogLog++) with sparser representation for low cardinalities (encode only the non-empty buckets in compressed form), 64-bit hashes throughout, and a more accurate bias correction empirically determined. Modern systems use HyperLogLog++ or further refinements.

## Where it shows up

- **Database engines**. Counting distinct values for query optimization, cardinality estimation in OLAP cubes. PostgreSQL, ClickHouse, Druid, BigQuery, Snowflake — all have HyperLogLog-based COUNT DISTINCT, often with a sketch you can persist and query later.
- **Network monitoring**. "How many unique source IPs hit this load balancer in the last 5 minutes?" Sketch per minute; merge for any window.
- **Ad analytics**. Unique users per ad campaign, per day, per region. Sketches per cohort; merges for any aggregation.
- **Search engines**. Distinct queries per topic; distinct terms per document corpus.

The killer requirement that drives HyperLogLog adoption is when you need to query distinct counts across *many* dimensions and *many* time windows. An exact approach would store all the elements, blowing up storage. HyperLogLog stores a constant-size sketch per cell; the sketches merge to any aggregation level.

## The lower bound

For estimating cardinality up to a multiplicative factor of \\(1 \pm \epsilon\\) with constant probability, the information-theoretic lower bound is \\(\Omega(1/\epsilon^2)\\) bits. HyperLogLog uses \\(O(\log\log n / \epsilon^2)\\) bits, with a small constant. Within a \\(\log\log n\\) factor of optimal — and the \\(\log\log n\\) is for the counter size, very small in practice (8 bits suffices for any cardinality you will encounter in a single Earth's worth of data).

Recent constructions (HyperLogLog without the \\(\log\log\\) factor, like the LogLog-Beta and Streaming HLL of Ertl 2017) push closer to the lower bound, with various trade-offs in implementation complexity. HyperLogLog remains the standard for production code because of its simplicity and battle-tested behavior.

## The wonder

The ratio of effective compression is the headline. A hash set storing a billion 16-byte items needs 16 GB. The corresponding HyperLogLog sketch is 12 KB. That is six orders of magnitude. The 12 KB version answers the count-distinct query within 1% relative error, and merges with other 12 KB sketches to count distinct across arbitrary partitions.

The construction is conceptually small: hash each element, look at the leading zeros, keep the bucketed max, harmonic-mean across buckets. You could explain it on a napkin. The fact that this gives a near-optimal cardinality estimator is a result of the mathematics of order statistics of geometric distributions, which the early designers of the algorithm had to prove out carefully. It is one of the strongest examples of an algorithm whose underlying idea is shockingly simple but whose correctness analysis is subtle.

## Where to go deeper

- Flajolet, Fusy, Gandouet, Meunier, *HyperLogLog: the analysis of a near-optimal cardinality estimation algorithm*, AofA 2007. The paper. Read alongside the original LogLog paper (Durand and Flajolet 2003).
- Heule, Nunkesser, Hall, *HyperLogLog in Practice*, EDBT 2013. Google's engineering improvements that made it production-grade.
