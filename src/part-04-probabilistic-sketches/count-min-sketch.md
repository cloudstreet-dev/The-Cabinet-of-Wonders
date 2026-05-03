# Count-Min sketch

You see a stream of \\(n\\) items, each tagged with some key. You want to know, at any moment, the count of any specific key. Exact: a hash map sized in proportion to the number of distinct keys. Count-Min: a small fixed-size 2D array whose dimensions you choose to bound the error; queries are constant-time; the sketch never grows beyond your chosen size, regardless of how many distinct keys appear.

The bound is one-sided: the sketch never *under*counts. It can over-count. The over-count is bounded with high probability and is small relative to the total stream size. For most streaming-counting problems — heavy-hitter detection, frequency estimation, anomaly detection — that is exactly the right shape.

## The construction

Allocate a 2D array of counters: \\(d\\) rows by \\(w\\) columns, all zero. Pick \\(d\\) independent hash functions \\(h_1, \dots, h_d\\), each mapping keys to \\(\{0, \dots, w-1\}\\).

To insert a key \\(x\\) (with optional weight \\(c\\), default 1):

\\[ \text{for each } i \in \{1, \dots, d\}: \quad \text{counter}[i][h_i(x)] \mathrel{+}= c \\]

To estimate the frequency of \\(x\\):

\\[ \hat{f}(x) = \min_i \text{counter}[i][h_i(x)] \\]

That is the whole sketch. \\(d\\) increments per insert, \\(d\\) reads per query.

```
        col 0   1     2     3     4     5     6     7
row 0:   3      9     2    12     0     7     5     4    <- h_0(x) hits this row
row 1:   0      8     6     1    11     3     7     2    <- h_1(x) hits this row  
row 2:   5      4     0     2     6    13     8     1    <- h_2(x) hits this row

query x: hash to columns (4, 2, 7)
         row counts: 0, 6, 1
         estimate = min = 0  (so x has not been seen)
```

Different keys may hash to the same column in any given row, contributing to that column's counter — that is the source of overestimation. The minimum across rows is the tightest estimate from the available evidence; it is at least the true count (since every inserted item incremented every cell in its hash positions) and is hopefully close to it.

## The error bound

Let \\(N\\) be the total weight of all inserts (sum of \\(c\\) values, equal to the number of inserts if all weights are 1). Let \\(\hat{f}(x)\\) be the sketch estimate for key \\(x\\), and \\(f(x)\\) the true count.

**Theorem (Cormode-Muthukrishnan 2005).** For \\(w = \lceil e/\epsilon \rceil\\) and \\(d = \lceil \ln(1/\delta) \rceil\\):

\\[ \hat{f}(x) \geq f(x) \quad \text{always.} \\]

\\[ \Pr[\hat{f}(x) \leq f(x) + \epsilon N] \geq 1 - \delta \\]

So the over-count, with probability at least \\(1 - \delta\\), is at most \\(\epsilon N\\). For \\(\epsilon = 0.001\\) and \\(\delta = 0.01\\), the sketch needs about \\(2719\\) columns and \\(5\\) rows — about 14000 counters total, say 56 KB if each is a 4-byte int. That sketches a stream of any size with overestimate of at most 0.1% of total stream weight, with 99% confidence.

The proof is a one-paragraph application of Markov's inequality applied to the per-row collision count, then independence across rows.

## Why one-sided is exactly right for heavy hitters

A *heavy hitter* is a key whose count exceeds some threshold (e.g., 1% of stream). Heavy hitters are typically what you care about: the few keys that dominate the stream.

Count-Min's overestimation property guarantees that any heavy hitter will appear with high estimated count. Light keys may appear with overestimated counts due to collisions, but their estimates are still bounded by their true count plus \\(\epsilon N\\), which is typically a small fraction.

To find heavy hitters, you can pair Count-Min with a small ordered list ("top-k" structure, like a min-heap of size \\(k\\)): for each incoming key, query the sketch, and if the estimated count exceeds the heap's minimum, insert into the heap (or update). The heap's contents are the candidate heavy hitters. The sketch handles the counting; the heap tracks the candidates.

This is the architecture of most real-time analytics on high-volume streams: network flows, ad impressions, search-query frequencies, log-line sources. Constant-memory, single-pass, and accurate for the heavy tail that matters.

## Comparison to related sketches

**Bloom filter**: yes/no membership; cannot count.

**Counting Bloom filter**: each cell is a counter, but you increment all hashed cells; query returns minimum (essentially the same as Count-Min with a single row, then \\(d\\) rows for accuracy). Count-Min is essentially a re-derivation of the same idea with a cleaner error analysis.

**Count-Sketch (Charikar, Chen, Farach-Colton 2002)**: closely related. Each row has \\(\pm 1\\) hash signs; the estimate is the *median* across rows (not min). Count-Sketch gives unbiased estimates with smaller error for typical key counts; Count-Min has the simpler one-sided bound and is used more often in practice. They have similar memory; the choice depends on the application.

**Misra-Gries**: a deterministic algorithm for heavy hitters, also constant memory but with weaker guarantees against adversarial streams. Often used together with Count-Min.

## Where it shows up

- **Network telemetry.** OpenSketch, Sonata, and other in-network telemetry frameworks use Count-Min for per-flow byte counts on routers with limited memory. A modern switch ASIC has Count-Min compiled into hardware.
- **Top-k queries in OLAP systems.** ClickHouse, Druid, others use approximate top-k, often Count-Min-based.
- **DDoS detection.** A sudden spike in count for a particular source IP is a Count-Min query.
- **Search engine query frequency.** Tracking the most popular queries in real time without storing every query.
- **Recommendation systems**. Approximate item-frequency tracking for stale-feature detection.

## What about deletions

Negative-weight inserts are allowed. The sketch can subtract counts. With deletions, the "minimum across rows" estimator no longer gives a one-sided bound — both over- and under-counting become possible, since some collisions can be negative. The Count-Sketch median estimator handles deletions better. In practice, "Count-Min Sketch with deletions" is an oxymoron and Count-Sketch is used.

## The wonder

The classical data structure for "count by key in a stream" is a hash map: O(distinct keys) memory. This grows unboundedly. For high-cardinality streams (every IP address in the world, every URL, every transaction) you cannot afford to keep them all.

Count-Min replaces an unbounded hash map with a fixed-size 2D array. Insertion is \\(O(d)\\), query is \\(O(d)\\), memory is \\(O(d \cdot w)\\). The error bound is sharp and one-sided, exactly matching the typical use case (heavy hitters dominate; light tail is unimportant). The construction is so simple you can re-implement it from memory, and it works.

The trade-off — control where the error goes — is the same as Bloom filters but in a different shape: there, false positives in a yes/no query; here, overestimates in a count query. The pattern, once you see it, recurs throughout the world of streaming sketches: pick the side of the error that matches the problem; ride that asymmetry to a much smaller data structure.

## Where to go deeper

- Cormode and Muthukrishnan, *An Improved Data Stream Summary: The Count-Min Sketch and its Applications*, Journal of Algorithms 2005. The original.
- Cormode, *Sketch Techniques for Approximate Query Processing*, in *Synopses for Massive Data*, 2011. Comparative survey of streaming sketches.
