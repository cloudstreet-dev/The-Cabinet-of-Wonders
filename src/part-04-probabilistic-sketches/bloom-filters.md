# Bloom filters

You can store a set of a million strings in 1.2 megabytes — about 10 bits per element — and answer "is this string in the set?" in constant time, with no false negatives, and a false positive rate you can dial in to whatever you want, say 1%. The structure cannot tell you what is *in* the set. It cannot enumerate. It cannot delete. But it can answer membership queries faster and with less memory than any data structure that does not have a probabilistic compromise.

Burton Bloom published it in 1970 and the construction has not really been improved on for the original problem. It is the textbook example of trading a small, controlled probabilistic error for orders-of-magnitude reductions in resources.

## The construction

Allocate a bit array of \\(m\\) bits, all initialized to zero. Pick \\(k\\) independent hash functions, each mapping strings to \\(\{0, 1, \dots, m-1\}\\).

To insert an element \\(x\\), compute \\(h_1(x), h_2(x), \dots, h_k(x)\\) and set those \\(k\\) bits in the array to 1.

To query whether \\(x\\) is in the set, compute the same \\(k\\) hashes and check whether all \\(k\\) bits are 1. If any is 0, \\(x\\) is definitely not in the set. If all are 1, \\(x\\) is *probably* in the set.

```
bit array (m=16):  [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]

insert("apple"):  h1, h2, h3 -> 2, 7, 13
                  [0 0 1 0 0 0 0 1 0 0 0 0 0 1 0 0]

query("apple"):   h1, h2, h3 -> 2, 7, 13   all 1 -> maybe in set (correct)
query("banana"):  h1, h2, h3 -> 4, 9, 13   bit 4 is 0 -> not in set (correct)
query("cherry"):  h1, h2, h3 -> 7, 2, 13   all happen to be 1 -> false positive
```

False positives happen when the bits for a query happen to coincide with bits set by other insertions.

## The math of false positives

Insert \\(n\\) elements. Each insertion sets \\(k\\) bits (with probability of collisions, but ignore that for now). The probability a specific bit is 0 after \\(n\\) insertions:

\\[ p_0 = \left(1 - \frac{1}{m}\right)^{kn} \approx e^{-kn/m} \\]

The probability that all \\(k\\) bits for a query of an absent element happen to be 1:

\\[ p_{\text{fp}} = (1 - p_0)^k = \left(1 - e^{-kn/m}\right)^k \\]

This false-positive rate is what you tune.

For fixed \\(m, n\\), differentiating with respect to \\(k\\) shows the optimal number of hashes is

\\[ k^* = \frac{m}{n} \ln 2 \\]

and at that optimum the false-positive rate is

\\[ p_{\text{fp}} = \left(\frac{1}{2}\right)^{k^*} = (0.6185)^{m/n} \\]

So for a 1% false positive rate, you need \\(m/n \approx 9.6\\) bits per element. For 0.1%, about 14.4. For 0.01%, about 19.2. The cost is logarithmic in the inverse error rate.

These numbers do not depend on the elements' size. The strings can be arbitrarily long; only the *count* enters. A Bloom filter of a million 100-byte strings uses the same space as a Bloom filter of a million 4-byte strings.

## Why no false negatives

Inserting an element sets bits to 1. Bits never get unset. A query of an inserted element finds the same bits and they are all 1. No false negative is possible (assuming the same hash functions are used).

The asymmetry — false positives possible, false negatives impossible — is exactly what you want for many applications. You use the Bloom filter as a fast pre-filter: "is this query worth doing the expensive lookup for?" If the filter says no, definitively skip. If it says yes, do the expensive lookup, which checks definitively.

## Where it shows up

- **Database query planning.** "Does this disk page contain the row I want?" Each row's existence-Bloom-filter is in memory. Skip pages whose filter says no.
- **CDN cache lookups.** Bloom filter of cached URLs to skip slow back-ends for known-misses.
- **Word-spell-check.** Bloom filter of valid words; flag any input not in the filter for closer review.
- **Crawlers and dedup.** "Have I already crawled this URL?" Approximate set membership saves billions of disk reads.
- **Bitcoin SPV clients (originally).** Send a Bloom filter of addresses you care about; full nodes return any transaction that matches. Privacy implications were quickly noticed; not a recommended use today.
- **Cache infrastructure.** Memcached, Redis modules, Cassandra, HBase, ClickHouse, ScyllaDB, RocksDB — every modern database with disk-resident data has Bloom filters at its core.

## Variants

**Counting Bloom filter.** Replace each bit with a small counter (4 bits each). Insert increments; delete decrements. Supports deletion at a cost of 4× the memory. False-positive math is similar.

**Cuckoo filter.** Different construction (see *Cuckoo filters* in this part). Supports deletion and is sometimes more memory-efficient than a counting Bloom filter at the same false-positive rate.

**Quotient filter and XOR filter.** Other modern variants with better cache locality on real hardware. The xor filter (Graf-Lemire 2019) achieves around 9.84 bits per element at 0.39% false-positive rate, slightly beating Bloom in memory and significantly in lookup speed.

**Scalable Bloom filter.** A sequence of Bloom filters with geometrically increasing capacity, used when you do not know \\(n\\) in advance. Insertion and query traverse all of them; false-positive rate is bounded by the sum.

## The lower bound

Carter et al. (1978) proved a lower bound: any *exact* set-membership data structure (no false positives, no false negatives) requires \\(\Omega(n \log(u/n))\\) bits to represent a set of \\(n\\) elements from a universe of size \\(u\\). For a universe of all 64-bit integers and \\(n = 10^6\\), that is about 44 bits per element.

A *probabilistic* membership structure with false-positive rate \\(\epsilon\\) requires at least \\(n \log_2(1/\epsilon)\\) bits in the lower-bound information-theoretic sense. Bloom filters use about \\(1.44 n \log_2(1/\epsilon)\\) bits — within a constant factor of optimal. Cuckoo and xor filters get closer to the lower bound.

## The wonder

You have a set. You want to query it for membership. The classical computer-science answer involves storing the elements in a hash table (about 8 bytes per element overhead in C++; much more in dynamic languages) or a balanced tree. The space is dominated by storing the elements themselves, plus structural overhead.

Bloom filters say: do not store the elements at all. Store a 1.2-megabyte array of bits, set a few bits per insertion, query by checking a few bits. Take a 1% false-positive rate as the price. You get a structure that is 50× smaller than a hash table, has the same constant-time query, and is correct in the only direction you actually care about (no false negatives).

The trade-off — a small, tunable, controlled error in exchange for a massive resource saving — is the prototype of all probabilistic-sketch wonders. Once you accept the framing, a flock of related sketches (HyperLogLog, Count-Min, MinHash, etc.) all become obvious shapes of the same trade.

## Where to go deeper

- Burton H. Bloom, *Space/Time Trade-offs in Hash Coding with Allowable Errors*, Communications of the ACM, 1970. Three pages.
- Broder and Mitzenmacher, *Network Applications of Bloom Filters: A Survey*, Internet Mathematics, 2004. Wide overview of where they showed up.
