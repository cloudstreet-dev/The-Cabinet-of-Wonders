# CRDTs

A team of writers in five different cities is editing the same document simultaneously. Each is offline; each types into their local copy; their copies do not communicate at all for hours. Then the network comes back and the copies sync. Without any central coordinator, without locking, without any user manually merging, the copies converge to the same final state — and that final state contains all the edits, in a globally consistent order. No edits are lost. No conflicts pop up. The merge is automatic, deterministic, and provably correct.

This is what a CRDT (Conflict-free Replicated Data Type) does. Shapiro, Preguiça, Baquero, and Zawirski formalized the framework in 2011, but the underlying idea — semilattice-based merge — predates that. CRDTs are the data structures behind Google Docs collaboration (their version), Figma's multiplayer editor, the latest end-to-end-encrypted real-time collaboration apps, and most modern offline-first sync systems.

## The setup

\\(n\\) replicas of some piece of state. Each replica accepts local updates and gossips them to others over an unreliable, possibly partitioned network. There is no central authority; no replica is special. Updates can arrive at different replicas in different orders.

The CRDT *invariant*: regardless of which order replicas receive each others' updates, they all converge to the same final state.

This is *strong eventual consistency*: if all replicas have received the same set of updates (in any order), they have the same state. No coordination is required. Replicas can apply updates as soon as they arrive.

## Two flavors

**State-based (CvRDT)**: each replica has a state, and replicas merge by taking some join operation on their states. The join must be a commutative, associative, idempotent operation (a semilattice). Replicas exchange whole states; the receiver merges incoming with local.

**Operation-based (CmRDT)**: replicas exchange individual operations. Each operation, when applied, must commute with concurrent operations from other replicas. The replicas eventually all see the same set of operations and apply them in any order; the result is the same.

The two models are formally equivalent (each can implement the other) but have different engineering trade-offs.

## A simple state-based CRDT: G-Counter

A grow-only counter (only increment). Each replica \\(i\\) has a vector \\([c_1, c_2, \dots, c_n]\\) where \\(c_j\\) is the count of increments performed at replica \\(j\\), as known to replica \\(i\\).

- **Increment** at replica \\(i\\): \\(c_i \mathrel{+}= 1\\).
- **Value**: \\(\sum_j c_j\\).
- **Merge** of two vectors \\(c, c'\\): elementwise max: \\([\max(c_1, c'_1), \max(c_2, c'_2), \dots]\\).

The merge is a join in the lattice of vectors-of-non-negative-integers. It is commutative, associative, idempotent. After replicas exchange and merge, every replica sees the same vector and the same total.

Two replicas might increment concurrently; their vectors disagree on each other's component. After merge, both have the max in each component, capturing both increments.

To support decrement: a *PN-Counter* uses two G-Counters, one for increments and one for decrements; the value is the difference.

## A more interesting CRDT: OR-Set

An add-and-remove set with the following non-trivial property: if Alice adds element \\(x\\) and Bob removes \\(x\\) concurrently, the final state has \\(x\\) — the add wins.

Implementation: each add tags the element with a unique ID (e.g., (replica-id, timestamp)). The state is a set of (element, ID) pairs. Removes track which IDs have been removed.

- **Add** \\(x\\): insert \\((x, \text{fresh-id})\\) into the set.
- **Remove** \\(x\\): record all current IDs of \\(x\\) in a "tombstones" set.
- **Value**: elements with at least one ID not in tombstones.
- **Merge**: take the union of (element, ID) pairs and the union of tombstones.

If Alice adds \\(x\\) (ID \\(a_1\\)) and concurrently Bob removes \\(x\\) (tombstone \\(b_0\\) for whatever ID was visible to Bob), the merge has \\((x, a_1)\\) and tombstone \\(b_0\\). \\(a_1\\) is not tombstoned, so \\(x\\) is in the set. The add survives.

This is "add-wins"; "remove-wins" variants exist by tagging removes with IDs and adding tombstone-or-not logic.

## CRDTs for sequences (text editing)

The hardest classical CRDT is a *sequence* — a list whose elements you can insert and delete at arbitrary positions, with concurrent insertions converging to the same order on all replicas. This is what real-time collaborative editors need.

The underlying problem: if Alice inserts "X" between positions 5 and 6, and Bob concurrently inserts "Y" between positions 5 and 6 in his copy, who comes first?

Approaches:

**RGA (Replicated Growable Array)**: each character has a unique ID and a "predecessor" reference. Order is determined by the predecessor relation, with ties broken by ID. Insertion is local; sync sends the new (ID, predecessor, content) triple.

**Treedoc**: positions are paths in a tree. Inserting between adjacent characters extends the tree downward. The tree's in-order traversal is the document.

**Logoot, LSEQ**: each character has a fractional position; inserting between positions creates a new fractional position. Distributed B-tree-like structure.

**Yjs's Yata, Automerge's text type**: modern CRDT-based text types optimized for memory and convergence speed.

The hard part of these is not the conceptual model but the engineering: a long-edited document accumulates tombstones and metadata. Modern CRDT libraries (Yjs, Automerge) handle gigabyte-scale documents with compact internal representations and careful garbage collection.

## What CRDTs cannot do

CRDTs make conflict-free merging *automatic*, but only because they encode "what should happen on conflict" in the data type. Some operations have no canonical conflict resolution:

- **Mutual exclusion**: "exactly one of these two updates should win" is not a CRDT property; it requires consensus.
- **Sequential constraints**: "X must happen before Y" cannot be enforced by a CRDT alone; both must be applied in some order.
- **Strong consistency**: CRDTs give *eventual* consistency, not strong. After local updates, my replica's value is correct *for me*; remote replicas catch up later.

For applications where these matter — banking, ticket booking, anything with hard ordering — CRDTs are insufficient and you need consensus (Paxos, Raft) or transactions.

For applications where eventual convergence is acceptable — collaborative editing, offline-first apps, distributed caches, presence indicators — CRDTs are perfect because they are *cheap*: no leader election, no quorum, no failures-during-vote scenarios. Replicas operate independently and converge when they can talk.

## Why this is different from "just merge with timestamps"

A naive "last-writer-wins by timestamp" can lose updates: if Alice and Bob both edit the same field at nearly the same time, the later timestamp wins and the earlier edit is silently dropped. CRDTs are designed so that *no information is lost in convergence*. Concurrent updates are *combined* (not replaced) according to the CRDT's defined merge semantics.

This is the key distinction. LWW merges are conflict-aware but lossy. CRDTs are conflict-free, by definition, because the merge operation produces a result that incorporates both inputs.

## Where they show up

- **Yjs, Automerge**: collaborative editor libraries powering apps like Notion (in part), Tldraw, and dozens of newer offline-first apps.
- **Riak, AntidoteDB**: databases whose value types are CRDTs.
- **Redis (modules)**: CRDT-based replication for active-active geo-distribution.
- **Git, in a sense**: version control's merge model is not strictly CRDT-based, but its merge-as-history-DAG approach is in the same family.

## The wonder

A few mathematical structures (semilattices, op-based-with-commuting-operations) capture exactly what is needed for *coordination-free* convergence. You define the merge operation as a join; you make sure operations commute; the data structure inherits convergence as a theorem.

The wonder is that this works at all for non-trivial data. Counters and sets are easy. Sequences (text editing) seem like they should be impossible — surely the order of concurrent insertions matters? — but with the right encoding (unique IDs, partial order on positions), they become CRDTs too. Modern collaborative-editor libraries are real, fast, and used by millions of people, with no central server arbitrating order.

The deeper wonder is that *coordination-freeness* is a real property to engineer for. In distributed systems with consensus, every operation has to round-trip with a quorum; this is expensive and breaks during partitions. CRDTs sidestep the consensus penalty entirely for operations that don't need it. The cost is figuring out which of your operations are commutative and idempotent enough to be CRDT-able. For an increasing fraction of applications (offline-capable apps, edge computing, peer-to-peer collaboration), the answer is "most of them."

## Where to go deeper

- Shapiro, Preguiça, Baquero, Zawirski, *Conflict-free Replicated Data Types*, INRIA Tech Report 2011. The defining paper.
- Kleppmann, *Designing Data-Intensive Applications*, Chapter 5. Production-engineer's view.
