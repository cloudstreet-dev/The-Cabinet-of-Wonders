# Cuckoo filters

Bloom filters can be improved on. Cuckoo filters are an approximate-membership data structure that supports deletions, has better cache locality on modern hardware, and at the same false-positive rate often uses *less* memory than a Bloom filter. The construction borrows from cuckoo hashing — each item has two possible bucket positions, and on insertion conflicts cascade through the table evicting and relocating until everything settles.

The wonder is that you can encode set membership using only a small *fingerprint* of each item (not the item itself, not even its full hash) and still resolve collisions exactly enough to support deletion.

## Cuckoo hashing first

A cuckoo hash table uses two hash functions \\(h_1, h_2\\). An item \\(x\\) can live in slot \\(h_1(x)\\) or \\(h_2(x)\\). To insert: try \\(h_1(x)\\); if empty, place there. Otherwise place in \\(h_2(x)\\); if empty, place there. Otherwise *evict* the existing item from one of the two slots, place \\(x\\) there, and re-insert the evicted item using its other slot. This cascades — the evicted item may displace another, and so on. With high probability the cascade terminates within \\(O(\log n)\\) steps and below the table's load factor; otherwise the table is rebuilt with new hash functions.

The resulting table has \\(O(1)\\) worst-case lookup (check at most two slots), \\(O(1)\\) expected insertion, and load factor up to ~50% (with two hashes; up to ~90% with multi-slot buckets).

## Cuckoo *filters* add fingerprints

In cuckoo *hashing*, each slot stores the item itself. In cuckoo *filtering*, each slot stores only a small fingerprint \\(f(x)\\) — say, 8 to 16 bits derived from a hash. The fingerprint is enough to answer membership queries: query \\(x\\), check whether \\(f(x)\\) appears at slot \\(h_1(x)\\) or slot \\(h_2(x)\\). If yes, "probably in set." If no, "definitely not in set."

False positives happen when another item has the same fingerprint and lives in the slot the query checks. The false-positive rate is roughly \\(2/2^F\\) for fingerprint size \\(F\\) — two slots checked, each with probability \\(2^{-F}\\) of fingerprint collision.

But cuckoo filters need a clever trick to support eviction without remembering items.

## The "partial-key" cuckoo hashing trick

Standard cuckoo hashing computes \\(h_2(x)\\) from the item itself. Cuckoo filtering only stores fingerprints, so it cannot compute \\(h_2(x)\\) from a fingerprint alone — the original item is unrecoverable.

Resolution: define the second slot from the first slot and the fingerprint:

\\[ h_2(x) = h_1(x) \oplus h_{\text{auxiliary}}(f(x)) \\]

where the XOR is on a hash of the fingerprint. This has the lovely property that \\(h_1(x) = h_2(x) \oplus h_{\text{aux}}(f(x))\\), so given a slot index and the fingerprint stored in it, the alternative slot is computable. Eviction can compute "where else can this fingerprint go" using only the fingerprint (which it has) and the current slot.

So during a cascade, when the filter evicts a fingerprint from slot \\(s_1\\), it places it at \\(s_2 = s_1 \oplus h_{\text{aux}}(f)\\), even though the original item is long gone. The trick is essential and is the cuckoo-filter contribution beyond plain cuckoo hashing.

## Insertion cascade

```
insert x:
  s1 = h1(x); s2 = s1 XOR h_aux(f(x))
  if slot s1 has empty space, store f(x) there
  else if s2 has empty space, store f(x) there
  else evict a random fingerprint y from s1 or s2,
       place f(x) there, then re-insert y at its
       alternative slot (which is the other of s1, s2 XOR h_aux(y))
       cascading until placement succeeds or
       max-kicks limit hits (rebuild)
```

With buckets of size 4 (each slot holds up to 4 fingerprints), load factors of 95%+ are achievable.

## Performance vs Bloom

For a target false-positive rate \\(\epsilon\\):

- Bloom: \\(\approx 1.44 \log_2(1/\epsilon)\\) bits per item.
- Cuckoo filter (8-bit fingerprint, ~95% load factor): about \\(8 / 0.95 \approx 8.4\\) bits per item, giving false-positive rate \\(\approx 2 \cdot 2^{-8} = 0.78\%\\). Compared to Bloom at the same rate (\\(\approx 1.44 \log_2 128 \approx 10\\) bits per item), 16% smaller.

For very low false-positive rates (under 0.001), Bloom can be more efficient because its bits-per-item scales as \\(\log(1/\epsilon)\\) while cuckoo's scales as \\(\log(1/\epsilon) + 3\\). The crossover is around \\(\epsilon = 10^{-3}\\).

Lookup performance: cuckoo filter checks two cache lines (the two bucket slots). Bloom filter checks \\(k\\) cache lines, one per hash function. Bloom is generally slower in cache-miss-dominated workloads, faster when cached.

## Deletion

To delete \\(x\\): compute fingerprint, try to find it in slot \\(h_1(x)\\) or \\(h_2(x)\\), remove if present.

Caveat: if \\(x\\) was *not* inserted but its fingerprint collides with an actually-inserted \\(y\\)'s fingerprint at the same slot, deleting \\(x\\) would incorrectly remove \\(y\\). Cuckoo filter deletion is correct only if you delete items that were actually inserted. (Calling delete on an item never inserted is undefined.)

This is more delicate than Bloom's lack of deletion altogether. For applications where you control inserts and deletes (a transient cache, a streaming window), it works fine.

## Variants

**XOR filters** (Graf, Lemire 2019): a different construction that achieves about 9.84 bits per item at 0.39% false-positive rate, slightly better than cuckoo at the same rate. Build is offline; insertions during use are not supported. Best for static datasets.

**Vacuum filter** (Wang et al. 2019): hybrid combining cuckoo's structure with better cache behavior; faster lookup than vanilla cuckoo.

**Quotient filter** (Bender et al.): an older alternative that uses run-length encoding of fingerprints in a sorted array. Cache-friendly, supports deletion, supports merging.

**Ribbon filter** (Dillinger 2021): linear-algebra-based; achieves close to information-theoretic optimal bits per item.

For most production purposes today, the choice between Bloom, cuckoo, xor, and ribbon filters comes down to whether you need deletion, dynamic insertion, and what false-positive rate you target.

## Where they show up

- **Cache and CDN systems.** Tracking which keys are in a hot tier; cuckoo filter's deletion support makes it easier to handle eviction.
- **Database engines.** RocksDB has a built-in option for ribbon filters; some systems use cuckoo filters for negative-lookup acceleration.
- **Network telemetry.** Cuckoo filters in OVS, Tofino-based programmable switches; lookup-and-delete behavior fits per-flow tracking.
- **Filesystem deduplication.** Modern dedup systems use approximate filters before exact lookup; cuckoo filters supports deletion as files are unlinked.

## The wonder

You can store a set in less space than a Bloom filter, support exact deletion (assuming honest workloads), have constant-time queries that touch at most two cache lines, and use only fingerprints — small derivatives of the items, not the items themselves. The construction depends on the unusual *partial-key cuckoo hashing* trick: defining the alternative slot in terms of the fingerprint and the current slot, so eviction can proceed without knowing the original item.

The trick is one of those where the simplicity is deceptive. Cuckoo hashing without partial-key alternation cannot have fingerprint-only storage, because the alternative slot would not be computable. The XOR is the entire mechanism that lets cuckoo filtering exist as a category. Without it, you have either cuckoo hashing (with full items, bigger storage) or Bloom filters (no deletion, sometimes more bits per item). With it, you have a third point on the design space that beats both for many workloads.

## Where to go deeper

- Fan, Andersen, Kaminsky, Mitzenmacher, *Cuckoo Filter: Practically Better Than Bloom*, CoNEXT 2014. The defining paper.
- Pagh, *Cuckoo Hashing*, Journal of Algorithms 2004. The underlying hashing technique.
