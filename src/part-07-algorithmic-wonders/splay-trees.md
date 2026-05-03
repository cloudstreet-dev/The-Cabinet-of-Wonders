# Splay trees

A binary search tree where every operation, from insertions to deletions to lookups, ends by rotating the accessed node all the way to the root. The tree therefore stays in a constantly-rebalancing state without any global rebalancing rule and without storing height or color information at each node. Worst-case operations cost \\(O(n)\\), but the *amortized* cost is \\(O(\log n)\\). And, conjecturally, splay trees are within a constant factor of *optimal* for any access sequence — they magically adapt to the workload.

Sleator and Tarjan published splay trees in 1985. The construction is one of the cleanest examples in computer science of a self-adjusting data structure that meets multiple competing optimality criteria, often with no per-node bookkeeping at all.

## The operation

Splay trees use a single primitive, *splay*, which moves a node to the root via tree rotations. After every search, insertion, or deletion, you splay the most recently touched node. That is the entire algorithm.

Splay uses three rotation cases, depending on the node's grandparent:

- **Zig** (no grandparent, just a parent): single rotation between node and parent. Used only at the root.
- **Zig-zig** (node and parent are both left children, or both right children): rotate parent with grandparent first, then node with parent.
- **Zig-zag** (node is a left child of right child, or vice versa): rotate node with parent, then with grandparent.

The crucial detail is *zig-zig*: rotate the *parent* first, *then* the node. The naive "rotate the node twice" gives a different shape — and a different (worse) amortized complexity. The exact rotation order matters, and Sleator and Tarjan figured it out.

```
Zig-zig (right-right):
       g                       x
      / \\                     / \\
     p   D     splay(x)       y   p
    / \\        =====>            / \\
   x   C                         z   D
  / \\                                  \\
 y   z                                   ... (rotated parts)
```

(Diagram approximate — the point is that the tree is reshaped to bring \\(x\\) to the root while shortening the path through \\(p\\).)

## The amortized analysis

Splay's operations have unbounded worst case (you can construct a tree that is essentially a linked list, then access the bottom). But amortized cost over any sequence of \\(m\\) operations on an \\(n\\)-node tree is \\(O(m \log n)\\).

The proof uses a *potential function*: assign each node \\(v\\) a *rank* \\(r(v) = \log s(v)\\) where \\(s(v)\\) is the size of the subtree rooted at \\(v\\). Define the tree's potential \\(\Phi\\) as \\(\sum_v r(v)\\).

The amortized cost of splaying \\(x\\) is \\(O(r(\text{root}) - r(x)) + 1 = O(\log n)\\). The proof is a careful case analysis of the three rotation patterns, showing that each rotation's actual cost (one or two unit operations) plus the change in potential is bounded by \\(3 \cdot \Delta r\\), where \\(\Delta r\\) is the change in the splayed node's rank during that rotation. Telescoping over all rotations in a splay gives \\(O(\log n)\\) amortized.

This is the *access lemma*. Once you have it, all splay-tree operations follow.

## Why no extra bookkeeping

Standard balanced trees (red-black, AVL, B-trees) maintain extra information per node — color bits, height counters, balance factors. Splay trees keep no extra information. Each node has just keys, values, and pointers to children. The balance is *implicit* in the tree's shape, which the splay operation maintains.

This matters in practice: splay-tree nodes are smaller and the algorithms are simpler. The downside is that splay-tree operations *modify* the tree even on lookups, which complicates concurrent access (every read is also a write).

## The conjectured optimality

Splay trees enjoy several remarkable properties:

**Static optimality**: For any access sequence drawn i.i.d. from some distribution \\(p\\), splay trees achieve, asymptotically, the entropy-bound expected access cost \\(O(\sum_i p_i \log(1/p_i))\\). This matches the optimal static binary search tree.

**Static finger property**: Accessing element with rank \\(r\\) (in sorted order), followed by accessing element with rank \\(r'\\), has amortized cost \\(O(\log |r - r'| + 1)\\). Locality is rewarded.

**Working set property**: Access to an element costs \\(O(\log w)\\) amortized, where \\(w\\) is the number of distinct elements accessed since the last access of this one. Recently-accessed elements are cheap.

**Sequential access**: Accessing every element in sorted order costs \\(O(n)\\) total — \\(O(1)\\) amortized per access, much better than \\(O(\log n)\\).

These properties are simultaneous: a single splay tree, with no parameters, automatically achieves all of them. No other balanced-search-tree structure has been shown to.

The **Dynamic Optimality Conjecture** (Sleator-Tarjan 1985): splay trees are within a constant factor of the optimal binary search tree for *every* access sequence — even adversarial ones. The conjecture has been open for 40 years. If true, splay trees are the universal optimal binary search tree.

Recent work (Wilber, Iacono, others) has shown that splay trees are at most an \\(O(\log \log n)\\) factor from optimal for various restricted classes of sequences. The full conjecture remains open.

## What it costs

The amortized cost is \\(O(\log n)\\). Worst-case is \\(O(n)\\). For real-time systems where worst-case latency matters, splay trees are unsuitable; AVL or red-black trees give worst-case \\(O(\log n)\\).

For workloads with locality — caches of any kind, code editors, network connection tables — splay trees match the workload exactly: hot keys live near the root, cold keys are deep but rarely visited. The amortized analysis ensures no surprises in long-run behavior.

## Where they show up

- **Sleator's data structure for path operations on trees** (link-cut trees) uses splay trees as the building block for representing tree paths. Solves the dynamic-tree problem in \\(O(\log n)\\) amortized.
- **Memory allocators**: tracking free lists and sizes. Splay trees give locality automatically.
- **Lookup caches in compilers, virtual machines, network stacks**: where access is highly skewed.
- **Editors and IDEs**: maintaining buffer-position indices. Cursor moves create access locality that splay trees exploit.

They are less common than red-black trees in standard libraries because the worst-case latency is hard to reason about. But for the right workload, they are remarkably efficient and remarkably simple.

## The wonder

A binary search tree with no balance information, no rebalancing rule beyond "always rotate to the root," achieves \\(O(\log n)\\) amortized cost on any operation, automatically adapts to access patterns, and is conjecturally optimal across *all* access sequences. The construction is short. The amortized analysis is clean. The list of optimality properties it provably satisfies (static optimality, working set, sequential access, finger property) is long.

The wonder is in the simplicity-vs-power trade-off. The simplest possible self-adjusting tree happens to be near-optimal for every workload. There is no theory that explains why this should be — just a long list of properties that happen to all be satisfied by the same construction. The Dynamic Optimality Conjecture, if eventually proved, will close the loop. For now, it is one of the standing-open problems in algorithmics, and splay trees are widely used in practice on the strength of their proven results plus the conjecture.

## Where to go deeper

- Sleator and Tarjan, *Self-Adjusting Binary Search Trees*, JACM 1985. The original.
- Iacono, *In Pursuit of the Dynamic Optimality Conjecture*, 2013 survey. Modern progress.
