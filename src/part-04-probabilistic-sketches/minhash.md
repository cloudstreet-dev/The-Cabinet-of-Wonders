# MinHash

You have a billion documents and you want to find pairs that are nearly duplicates — say, 80% similar by content. The pairwise comparison approach is \\(\binom{10^9}{2}\\) — half a quintillion comparisons — which never finishes. MinHash gives you a 200-byte signature per document such that the *Jaccard similarity* of any two documents can be estimated to within 1% just by comparing the signatures, and a small modification turns this into a sub-quadratic similarity-search algorithm.

The MinHash signature is the minimum hash of the document's set under each of \\(k\\) independent hash functions. Stored as \\(k\\) numbers, queried by counting matching positions. The structure is so simple that it slipped into wide deployment immediately after Broder published it in 1997, when AltaVista needed to detect near-duplicate web pages.

## Jaccard similarity

For two sets \\(A, B\\):

\\[ J(A, B) = \frac{|A \cap B|}{|A \cup B|} \\]

Ranges from 0 (disjoint) to 1 (identical). For documents represented as sets of shingles (\\(k\\)-character substrings, or \\(k\\)-word phrases), Jaccard similarity is a useful proxy for content similarity.

The exact computation requires both sets in memory. For two documents totaling 100 KB of text each, that is 100 KB * 2 = 200 KB just to compare. Doable for one comparison; intractable for billions.

## The MinHash trick

Pick a hash function \\(h\\) that maps elements of the universe to integers (or reals in [0, 1]) uniformly at random. For a set \\(S\\), define

\\[ \text{minhash}_h(S) = \min_{s \in S} h(s) \\]

The astonishing property:

\\[ \Pr_h[\text{minhash}_h(A) = \text{minhash}_h(B)] = J(A, B) \\]

The probability that two sets have the same minhash, under a uniformly chosen hash function, is exactly their Jaccard similarity.

## The proof

Consider the elements of \\(A \cup B\\), each with a uniformly random hash value. The element with the smallest hash value is the *unique minimum* of \\(A \cup B\\). It is in \\(A\\) iff \\(\text{minhash}(A) \leq \text{minhash}(B \setminus A)\\), but since the minimum of \\(A \cup B\\) is the smallest hash overall, and it is in some specific element, the element of \\(A \cup B\\) with the smallest hash:

- Is in \\(A \cap B\\): then \\(\text{minhash}(A) = \text{minhash}(B)\\).
- Is in \\(A \setminus B\\): then \\(\text{minhash}(A) < \text{minhash}(B)\\).
- Is in \\(B \setminus A\\): then \\(\text{minhash}(A) > \text{minhash}(B)\\).

The first case happens with probability \\(|A \cap B| / |A \cup B| = J(A, B)\\) by uniformity of the hash. Done.

## The signature

A single minhash is one bit of information ("equal or not"); not very useful. Use \\(k\\) independent hash functions and stack them: the *signature* of \\(S\\) is

\\[ \text{sig}(S) = (\text{minhash}_{h_1}(S), \text{minhash}_{h_2}(S), \dots, \text{minhash}_{h_k}(S)) \\]

a vector of \\(k\\) numbers (or \\(k\\) bits, if you take \\(\text{minhash}_h \mod 2\\)).

Estimate Jaccard similarity by comparing signatures position-wise: the fraction of positions where two signatures agree is an unbiased estimator of \\(J(A, B)\\). Standard deviation is \\(\sqrt{J(1-J)/k}\\); for \\(k = 200\\) and \\(J = 0.8\\), the standard deviation is about 2.8% relative.

So 200 hashes per document give Jaccard estimates with low-percent error. Storage is 200 numbers per document. Query is 200 comparisons.

## Sub-quadratic search via LSH

Even with 200-number signatures, comparing every pair is still \\(O(n^2)\\) signatures comparisons. For \\(n = 10^9\\), still too many.

Locality-Sensitive Hashing (LSH) buckets signatures so that similar signatures fall into the same buckets with high probability and dissimilar signatures with low probability.

For MinHash signatures, the LSH scheme: divide the \\(k\\)-position signature into \\(b\\) bands of \\(r\\) rows each (\\(k = b \cdot r\\)). For each band, hash the \\(r\\) values in that band into a bucket. Two documents are *candidate* near-duplicates if they collide in *at least one* band.

The probability that two documents with Jaccard similarity \\(s\\) collide in a specific band is \\(s^r\\) (all \\(r\\) positions must agree). The probability they collide in at least one of \\(b\\) bands is

\\[ 1 - (1 - s^r)^b \\]

This curve has an "S-shape": low for small \\(s\\), nearly 1 for large \\(s\\), with a sharp transition near \\(s^* = (1/b)^{1/r}\\). Tune \\(b\\) and \\(r\\) to put the transition where you want — typically a 0.7 or 0.8 threshold for near-duplicate detection.

```
P(collide)
 ^
1 |                  ___________
  |                 /
  |                /
  |               /  <-- sharp transition near s*
  |              /
  |             /
  |   _________/
0 +--------------------------> Jaccard similarity
  0       s*                 1
```

Search becomes: hash each signature into \\(b\\) buckets. For each query, look up its buckets and only compare against signatures that share a bucket. Expected number of candidates is much smaller than \\(n\\), often by orders of magnitude.

## What it shows up in

- **Web-scale near-duplicate detection.** AltaVista, Google, Bing all used MinHash + LSH for crawl deduplication. Spider hits a page; if its MinHash signature collides in any band with a previously-crawled page, compare in detail.
- **Plagiarism detection.** Turnitin and similar services do MinHash on chunks of student documents against a corpus of references.
- **Recommendation systems.** Find users with similar item-history sets. Item-set Jaccard similarity is a standard input for collaborative filtering.
- **Clustering.** Single-linkage clustering on document collections, where the similarity graph is built from MinHash-LSH candidates.
- **Genome analysis.** \\(k\\)-mer sets for sequencing reads can be MinHashed for fast similarity search; the Mash tool uses this for taxonomic classification.

## Variants

**MinHash with one hash and bottom-\\(k\\)**: instead of \\(k\\) independent hashes, use one hash and keep the \\(k\\) smallest values. Statistically nearly equivalent, slightly biased, but cheaper to compute (one hash per element instead of \\(k\\)). Used in *MinHash sketches* and the Mash tool above.

**Weighted MinHash**: extends to multisets and fractional weights. Different constructions (Manasse et al., Ioffe). Harder to implement; needed for, e.g., word-frequency histogram comparison.

**Densified MinHash**: a recent technique to reduce variance further by reusing hash values across positions. Subtle but important in production engines.

## Why this is different from Bloom

Bloom filter answers "is \\(x\\) in \\(S\\)?". MinHash answers "how similar are \\(S\\) and \\(T\\)?". They are different problems, but both substitute a small probabilistic sketch for an explicit data structure. The cost trade-off — small fixed-size sketches in exchange for a controlled probability of error — is the same shape; the operation supported is different.

## The wonder

You compress two documents to 200 numbers each. Comparing those numbers element-wise tells you, with low error, what fraction of their content they have in common. The compressed representation discards almost everything about the documents — their order, their structure, their content. What survives is the Jaccard similarity, encoded entirely in the empirical distribution of the minimum of independent random hash functions.

The fact that this works at all rests on a single combinatorial identity: the minimum of two sets under a random hash equals the minimum of their union, with probability proportional to their intersection. A line of probability gives you a structure that detects near-duplicates among billions of documents on commodity hardware. That is real: MinHash is a workhorse in production systems handling the entire indexed web.

## Where to go deeper

- Andrei Broder, *On the Resemblance and Containment of Documents*, Compression and Complexity of Sequences 1997. The original paper.
- Leskovec, Rajaraman, Ullman, *Mining of Massive Datasets*, Chapter 3 (free online). Modern textbook treatment with LSH variants.
