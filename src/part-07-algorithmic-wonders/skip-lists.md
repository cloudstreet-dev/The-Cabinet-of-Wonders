# Skip lists

A linked list with multiple "express lanes" laid on top, where each next-level lane is a randomly selected sparser subset of the level below. Search descends from the top lane down, skipping over many elements at each step. The result is \\(O(\log n)\\) expected time for search, insert, and delete — matching balanced binary search trees — without ever doing any rebalancing. The structure stays balanced, in the probabilistic sense, simply because each new element flips a coin to decide how high it should reach.

Pugh published it in 1990 as a deliberate alternative to balanced BSTs: same asymptotic guarantees, dramatically simpler code, and no parental tree-rotation pain. Redis's sorted-set primitive uses a skip list. The kernel scheduler in Linux uses one for the runqueue.

## The structure

A skip list of \\(n\\) keys consists of multiple linked lists stacked vertically:

- Level 0: a sorted linked list of all keys.
- Level 1: a subset of keys, each independently included with probability 1/2.
- Level 2: a subset of level 1, again with each included with probability 1/2. So each element of level 0 is at level 2 with probability 1/4.
- ... and so on, up to roughly \\(\log_2 n\\) levels.

Each node has multiple "next" pointers, one per level it appears in. Each pointer at level \\(k\\) skips to the next node that also appears at level \\(k\\) — typically several nodes ahead at higher levels.

```
                 ___________________________________
level 3:        |                                   |
                 1 ----------------------> 17 -------> nil
                                            |
level 2:        1 -------> 7 -------> 17 -------> 25 -> nil
                |          |          |           |
level 1:        1 -> 4 -> 7 -> 12 -> 17 -> 21 -> 25 -> 30 -> nil
                |    |    |    |     |     |     |     |
level 0:        1 -> 4 -> 7 -> 12 -> 17 -> 21 -> 25 -> 30 -> nil
```

Searching for 21:
1. Start at top level (3) at the head. Look right: 17 < 21, go right. Look right again: nil. Go down.
2. Now at level 2 at node 17. Look right: 25 > 21. Go down.
3. Now at level 1 at node 17. Look right: 21. Found.

The search descends a stair-step pattern: at each level, walk forward until the next node is too big, then drop down. Expected depth is \\(O(\log n)\\); expected forward steps per level is \\(O(1)\\); total work \\(O(\log n)\\) expected.

## Insertion

Pick a random "level" for the new node by flipping coins:

```
level = 0
while flip_coin() == heads:
    level += 1
```

This gives a geometric distribution: P(level = \\(k\\)) = \\(2^{-(k+1)}\\). Cap at \\(\log n\\) for sanity.

Search to find the position; remember the predecessors at each level visited. Splice the new node into all levels up to its random level.

\\(O(\log n)\\) expected. No rebalancing.

## Deletion

Search to find the node. Splice it out of all levels at which it appears. \\(O(\log n)\\) expected.

## Why this stays balanced

A balanced BST gets balanced by explicit rebalancing rules (rotations on insert/delete to maintain height invariants). Skip lists get balanced by *random level selection*: the average density at each level is half of the level below, so the heights stay logarithmic by the law of large numbers.

There is no worst-case input that breaks a skip list (assuming the level-selection coin is not adversarial). The randomization is at insertion time, so even an adversarially-ordered input sequence produces a balanced structure with high probability.

The probability of the search taking more than \\(c \log n\\) time, for some constant \\(c\\), is exponentially small. It is one of the cleanest randomized data structures, with simple analysis.

## Why this is engineering-friendly

**Code complexity**: a skip list is roughly 30-50 lines of code in any language. A red-black tree is 200-400. The simplicity matters for verification, debugging, and porting.

**Concurrent operation**: skip lists are easier to make concurrent than balanced BSTs. Multiple threads can search/insert/delete with fine-grained locking on a per-node basis, since rebalancing is local. Lock-free skip lists exist (Fraser, Harris) and are used in production. ConcurrentSkipListMap in Java is a popular implementation.

**No worst-case rebalancing pause**: insertions never trigger a cascading rebalance. The work per insertion is bounded by \\(O(\log n)\\) without amortization.

**Cache behavior**: the linked-list-with-multiple-pointers structure is less cache-friendly than B-trees (which have wide nodes). For in-memory workloads, B-tree variants are often faster in absolute terms.

## Where they show up

- **Redis**: sorted sets (ZSET) use a skip list combined with a hash table. The skip list keeps the elements ordered; the hash gives \\(O(1)\\) access by member name. Redis's skip lists support range queries, which are easy to implement on a sorted-list-with-express-lanes structure.
- **LevelDB, RocksDB**: in-memory write buffer is a skip list (`MemTable`). Provides ordered iteration plus \\(O(\log n)\\) inserts.
- **Linux kernel**: the SCHED_DEADLINE scheduler uses skip lists for the deadline-ordered runqueue.
- **Java's ConcurrentSkipListMap**: lock-free concurrent ordered map, widely used.
- **Cassandra**: in-memory data structures.

## A pleasant variant: deterministic skip lists

Munro, Papadakis, Sedgewick (1992) showed how to make a skip list with strictly bounded heights — no randomization, deterministic worst-case \\(O(\log n)\\). The trick is to maintain at each level a constraint like "no more than 3 consecutive elements at the same level," and enforce it on insertion. The data structure becomes more complex but loses the probabilistic guarantees in favor of deterministic ones. Less commonly used in practice, but theoretically interesting.

## The wonder

A balanced data structure where the balance is provided by *coin flips at insertion time*, with no rebalancing ever needed. The asymptotic guarantees match red-black trees (\\(O(\log n)\\) for everything), and the implementation is dramatically simpler.

The implicit lesson: for many data-structure problems, you can replace deterministic balancing with random level assignment and get the same guarantees. The randomization moves the work from "after the structure is modified" to "at the moment of modification, decide how the new node should fit in." The latter is local; the former (BST rebalancing) is global. Local randomization replaces global determinism, and the result is simpler code that performs as well asymptotically.

This is a recurring pattern in randomized data structures: treaps, randomized binary search trees, even hashing itself. Determinism is often more elaborate than the equivalent randomized version, with only a small constant-factor cost. Skip lists are perhaps the cleanest example.

## Where to go deeper

- William Pugh, *Skip Lists: A Probabilistic Alternative to Balanced Trees*, CACM 1990. The original.
- Pugh, *Concurrent Maintenance of Skip Lists*, technical report 1990. The early concurrent variants.
