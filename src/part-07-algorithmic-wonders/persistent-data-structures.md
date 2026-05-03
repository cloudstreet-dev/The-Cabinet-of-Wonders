# Persistent data structures

A data structure where every "modification" returns a new version, and all old versions remain valid and accessible. Updating a million-element list returns a new million-element list in \\(O(\log n)\\) time and \\(O(\log n)\\) extra space — not \\(O(n)\\). The old list is unchanged. Both are usable. The naive intuition that "you must copy everything to keep the old version" is wrong; structural sharing makes it cheap.

This is the data-structure foundation of functional programming, as deployed in Clojure, Scala (immutable collections), Haskell, OCaml, Erlang, Elixir, and recently in Rust's `im` crate and JavaScript's Immutable.js. It is also the secret behind why Git can store a hundred snapshots of a million-line repo without storing a hundred million lines.

## What "persistent" means here

Two senses worth distinguishing:

- **Partial persistence**: any past version can be queried, only the current version can be modified.
- **Full persistence**: any past version can be modified, producing a new version that branches off.
- **Confluent persistence**: two versions can be merged into a new one (Git's territory).

In functional programming, the default is *full persistence*: any version can be the "input" to any operation, producing new versions. Older versions are not invalidated.

## The wrong way

Naive persistent list (in pseudocode):

```python
def update(lst, i, v):
    new_lst = list(lst)  # copy entire list
    new_lst[i] = v
    return new_lst
```

\\(O(n)\\) per update. Memory usage proportional to the number of updates times the size. Hopeless.

## The right way: structural sharing

A persistent linked list (cons-cell-based):

```
head -> [a] -> [b] -> [c] -> [d] -> nil

prepend e: returns
new_head -> [e] -> [a] -> [b] -> [c] -> [d] -> nil

The new head shares the tail with the original.
Both lists are valid; both are O(1) to construct and O(n) to traverse.
```

Prepending an element to a list takes \\(O(1)\\) time and \\(O(1)\\) extra space — just one new cons cell. Both the old list and the new list are valid; they share the same tail.

This is the magic of immutability: if you cannot mutate, sharing is free. A new version that wants to differ in the front of the list keeps a pointer to the rest of the old list. No copying.

Linked lists give cheap *prepend* but only \\(O(n)\\) random access. To get fast random access *and* cheap updates, you need a tree-shaped structure.

## Persistent vectors via tree

Clojure's `PersistentVector` uses a tree of fan-out 32. A vector of \\(n\\) elements is a tree of depth \\(\lceil \log_{32} n \rceil\\) — for billion-element vectors, depth at most 6. Random access is \\(O(\log_{32} n) = O(1)\\) for any practical \\(n\\).

To update element \\(i\\): walk down the path from root to leaf \\(i\\) (6 nodes), and copy each node along that path with the appropriate child pointer changed. The other branches are shared with the old version.

```
       root
      /    \\
     A      B       <- shared branches
    / \\    / \\
   ...  ... ... ... <- update changes one leaf and copies the path of 6 nodes
                     up to the root; everything else is shared.
```

Update cost: \\(O(\log n)\\) time, \\(O(\log n)\\) extra space (six new nodes, plus the new leaf). Old vector still valid. Both usable.

This is the basis of every persistent vector / list / sequence in modern functional languages.

## Persistent maps via Hash Array Mapped Trie (HAMT)

For maps and sets, the standard structure is the *Hash Array Mapped Trie* (Bagwell, 2001):

- A trie indexed by chunks of the hash of the key (5 or 6 bits per level).
- Each interior node has up to 32 (or 64) children, stored as a sparse array indexed by a bitmap.
- Leaves contain key-value pairs.

Lookup: hash the key, walk down the trie chunk by chunk, find the leaf. \\(O(\log_{32} n) = O(1)\\) for practical \\(n\\).

Update: walk down, copy the path, return new root. Same structural-sharing pattern as persistent vectors. \\(O(\log n)\\) time and space; old version preserved.

Clojure, Scala, Rust's `im`, JavaScript's Immutable.js, and ClojureScript all use HAMTs for their persistent maps. A 64-bit hash and 5-bit chunks gives a trie of depth at most 13; for any realistic dataset, lookup and update are essentially constant-time.

## Other persistent structures

**Finger trees** (Hinze and Paterson 2006): persistent sequences with amortized \\(O(1)\\) head/tail access on both ends, \\(O(\log n)\\) random access, \\(O(\log n)\\) split/concat. Used in Haskell's `Data.Sequence`.

**Red-black trees, AVL trees, weight-balanced trees**: classical balanced BSTs adapted to persistence. Path copying gives \\(O(\log n)\\) per operation. Used in OCaml's `Map`, Scala's `TreeMap`.

**Persistent priority queues**: leftist heaps, splay heaps, pairing heaps — all have persistent variants.

**Persistent disjoint-set forests**: harder, but Driscoll-Sarnak-Sleator-Tarjan (1989) showed how to make any pointer-based data structure persistent with \\(O(1)\\) overhead per operation, using *fat nodes* (nodes that record their version-history) or *path copying*. The general theory.

## Why this is the foundation of functional programming

Functional programming insists on immutable values. To compute, you produce new values from old ones — never modifying. The whole edifice would be inefficient if "produce a new value" meant "copy everything." With persistent data structures, "produce a new value" is cheap, and the language's semantics (referential transparency, easy reasoning, parallelism without locks) are realizable in practice.

Clojure shipped persistent data structures as the language's *default* in 2007. Every built-in collection — list, vector, map, set — is persistent. Mutation is opt-in via separate transient or atom types. The performance is good enough that Clojure code is competitive with Java code that uses mutable collections, for most workloads.

## What this enables

**Time travel debugging.** Save the state at every step of a computation; the state is a tiny pointer to a persistent structure. Step backwards by reverting to a saved pointer.

**Undo/redo without effort.** The undo stack is a stack of versions. Re-doing is just dereferencing.

**Cheap branching.** A code editor with multiple buffers, all sharing most of their content, costs \\(O(\text{distinct lines})\\) memory. Git is essentially this for source files.

**Optimistic concurrency.** Multiple threads can read and modify "their own" version of the data; merging is by re-applying operations to the latest version. (CAS-based atomic-pointer updates make this lock-free.)

**Pure functions in a stateful language.** A function that "modifies" its argument actually returns a new version; the caller can choose to use the new or old. Nothing is mutated outside of explicit assignment.

## The data-structural cost

Persistent structures have small constant-factor overhead vs. mutable structures: a persistent vector update is 5-10× slower than a mutable array update; persistent map lookup is 2-3× slower than a hash table. The trade-off is the absence of synchronization (immutable structures are inherently thread-safe), the absence of aliasing bugs (no one else modifies your data), and the cheapness of saving versions.

For workloads where sharing-of-versions matters more than per-update speed (collaborative editing, undo systems, compiler symbol tables, version-control-like systems), persistent structures are not just convenient — they are the right answer asymptotically.

## The wonder

The intuition that "you have to copy everything to keep an old version around" is wrong, and it is wrong by an exponential margin. A version of a million-element data structure can differ from another by a few elements yet share the rest at \\(O(\log n)\\) memory cost. The two versions are independently mutable (functionally), behaviourally, in every respect — they just happen to share most of their internal nodes.

The right data-structural shape — a tree with structural sharing — turns "preserve all history" from a quadratic cost into a logarithmic one. After two decades, this is one of the most influential ideas in language design: it is the reason Clojure exists, why Git scales, why Erlang and Elixir handle concurrency without locks, and why React's reconciliation works.

## Where to go deeper

- Okasaki, *Purely Functional Data Structures*, Cambridge 1998. The textbook.
- Bagwell, *Ideal Hash Trees*, EPFL 2001. The HAMT paper.
- Driscoll, Sarnak, Sleator, Tarjan, *Making Data Structures Persistent*, JCSS 1989. The original general theory.
