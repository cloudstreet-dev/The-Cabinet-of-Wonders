# Cuckoo hashing

A hash table where every key has *exactly two* possible slots, and lookups always check those two slots and only those two. No probing, no chains, no skipping over deleted entries. Worst-case lookup is two memory accesses. Insertion may need to relocate the previous occupant — kicking it out, "cuckoo-style," to its other slot — but this cascade resolves quickly with high probability, and the resulting structure has constant-time operations in the worst case.

Pagh and Rodler published it in 2001. It is the cleanest hash table construction in the worst-case sense: lookups are \\(O(1)\\) — not amortized, not expected, but worst case.

(See also *Cuckoo filters*, which apply the same idea to approximate-membership filtering.)

## The basic two-table version

Two arrays \\(T_1, T_2\\) each of size \\(m\\). Two hash functions \\(h_1, h_2\\). A key \\(x\\) lives in either \\(T_1[h_1(x)]\\) or \\(T_2[h_2(x)]\\) — never both, and never anywhere else.

**Lookup** of \\(x\\): check both possible slots. If \\(x\\) is in either, return it. If not, key is absent.

**Insert** of \\(x\\):
1. Try \\(T_1[h_1(x)]\\). If empty, place \\(x\\) there. Done.
2. Else, evict the key \\(y\\) currently in \\(T_1[h_1(x)]\\). Place \\(x\\) there.
3. Try to insert \\(y\\) at \\(T_2[h_2(y)]\\). If empty, place. Done.
4. Else, evict the key currently there. Continue cascading.

If the cascade reaches a length cap (say, \\(O(\log n)\\)) without resolving, *rehash*: pick new hash functions and rebuild. With load factor below the threshold (\\(\sim 0.5\\) for two-table cuckoo), rehashes are exponentially rare.

```
insert x:
  if T1[h1(x)] empty: place x there.
  else:
    y = T1[h1(x)]; T1[h1(x)] = x; current = y; current_table = 2
    loop:
      slot = (current_table == 1 ? T1[h1(current)] : T2[h2(current)])
      if slot empty: place current there. done.
      else:
        evict_key = slot's contents; place current; current = evict_key
        flip current_table
        if iterations > limit: rehash.
```

## Why it works

The capacity question: when does the eviction chain terminate?

Model: each key \\(x\\) is a node in a *bipartite multigraph* with vertex set \\(T_1 \cup T_2\\) and edge \\(\{h_1(x), h_2(x)\}\\) representing one of \\(x\\)'s possible homes. With \\(n\\) keys and \\(2m\\) slots, this is a random multigraph with \\(n\\) edges among \\(2m\\) vertices.

For load factor \\(n/(2m) < 1/2\\), the random graph contains no cycle (with high probability). Each connected component has at most one cycle's worth of "tightness," and inserting a new key extends a path that terminates at a free slot. So the cascade has length \\(O(\log n)\\) with high probability.

For load factor \\(\geq 1/2\\), the graph almost certainly contains cycles, and insertion can cycle forever. The threshold for two-table cuckoo is exactly 1/2 in the limit.

## Variants

**\\(d\\)-ary cuckoo hashing**: each key has \\(d\\) possible slots (3-ary, 4-ary). Higher \\(d\\) gives higher tolerable load factor. 4-ary cuckoo achieves load factor ~97%. The cascade is more complex but still terminates with high probability.

**Bucketed cuckoo**: each slot holds \\(b\\) keys (a bucket). 2-table 4-bucket cuckoo achieves load factor ~95% and has good cache behavior (each slot is one cache line). This is the practical workhorse.

**Cuckoo with stash**: a small auxiliary array (the *stash*) holds keys that fail to find a home. Reduces the chance of rebuild and improves load factor.

## Where it shows up

- **Memcached, Redis (alternatively)**: cuckoo-based hash tables are an alternative to chained hashing for high-throughput in-memory key-value stores.
- **DPDK and high-performance network packet processing**: cuckoo tables for flow lookup, where worst-case lookup time matters for predictable latency.
- **Compilers and JIT runtimes**: symbol tables, compile-time hash tables.
- **CPU TLB lookup engines**: some hardware uses cuckoo-style multi-table lookups for translating virtual addresses.

## The lookup-time win

Linear-probing hash tables can have lookup time of \\(O(\sigma^2)\\) on average for load factor \\(\sigma\\), with bad cache behavior near saturation. Chained hash tables have \\(O(1)\\) average but the chains can be long. Both have worst case \\(O(n)\\) for adversarial inputs.

Cuckoo: \\(O(1)\\) worst case for lookup. Two cache lines, always. The cascade only happens on insert, and rebuild is rare. For workloads that are mostly read (caches, lookup tables), cuckoo's predictable performance is appealing.

## What about adversarial inputs

Hash tables under adversarial input (where the adversary chooses keys to maximize collisions) have been a security issue (algorithmic complexity attacks). Cuckoo hashing is sometimes claimed to be more resistant, but actually: an adversary who knows your hash functions can construct keys that all hash to the same two slots, forcing a rebuild. So cuckoo is *not* automatically safe against algorithmic-complexity attacks.

The standard defense — rotate hash function seeds at startup, use cryptographic hash functions — applies to cuckoo hashing too, and is essential.

## Implementation challenges

The cascade complicates concurrent updates. Multiple insertions may try to evict the same key in different directions. Lockless cuckoo hashing (Li-Andersen, 2014) uses optimistic concurrency control: readers retry if a write is detected during their lookup; writes use versioned cells to avoid blocking readers.

Real implementations also have to handle the rebuild cost. If you size the table conservatively (load factor < 1/2), rebuilds are essentially never needed in normal operation. If you size aggressively, you accept the occasional latency spike.

## The wonder

A hash table whose lookup cost is *literally* two memory accesses, in the worst case, regardless of load factor. Insertion has a slightly more complex path but resolves with high probability in constant time. The construction is more elegant than chained hashing or quadratic probing — there is no probe sequence to worry about, no auxiliary data to maintain, no degraded performance under load (until you hit the threshold and rebuild).

The trick is the underlying random-graph structure. Each key creates an edge between its two possible homes. As long as the random graph is sparse, components are simple, cascades are short. The sparsity threshold (load factor 1/2 for two-table) is precisely the threshold at which random graphs become connected and cyclic; cuckoo hashing inherits its capacity from random-graph theory.

It is one of the cleanest examples of a non-obvious algorithm whose correctness analysis is not "case analysis" but rather "this object behaves like this random structure" — borrowing intuition from a well-developed theory of random graphs to bound the cost of a data-structure operation.

## Where to go deeper

- Pagh and Rodler, *Cuckoo Hashing*, Journal of Algorithms 2004 (preliminary version: ESA 2001). The original.
- Mitzenmacher, *Cuckoo Hashing With a Stash*, ESA 2008. Practical improvements.
