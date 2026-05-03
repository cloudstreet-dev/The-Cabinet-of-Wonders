# Merkle trees

You publish a single 32-byte hash. From that hash alone, anyone can later verify the contents of any specific entry in a database of a billion records, given the entry plus a 30-element proof. They never need the rest of the database. They cannot be tricked into accepting a wrong record. The original 32 bytes commits the publisher to every record in the database, simultaneously, all the way down.

Ralph Merkle described this in his 1979 thesis. It is the structural skeleton of every blockchain, every git repository, every distributed-storage integrity protocol. It is one of the simplest constructions that does something the unaided intuition does not believe possible.

## The construction

Take \\(n\\) data items \\(d_1, d_2, \dots, d_n\\) (assume \\(n\\) is a power of 2; pad if not). Hash each one with a cryptographic hash function \\(H\\) (SHA-256, BLAKE3, etc.):

\\[ h_i^{(0)} = H(d_i) \\]

These are leaves at level 0. Pair them up and hash each pair to get level 1:

\\[ h_i^{(1)} = H(h_{2i-1}^{(0)} \| h_{2i}^{(0)}) \\]

Repeat: each level halves the count, until one hash remains. That last hash is the *Merkle root*.

```
                root
               /     \
             ab        cd
           /    \    /    \
          a      b  c      d
          |      |  |      |
          d_1   d_2 d_3   d_4
```

The root commits to all leaves. Change any leaf, the root changes. The root is the entire commitment — 32 bytes, regardless of how many leaves.

## The proof of inclusion

To prove that \\(d_3\\) is in the tree, present:

- The leaf \\(d_3\\) and its leaf hash \\(c = H(d_3)\\).
- The sibling on each level along the path from \\(d_3\\) to the root: \\(d, ab\\) in this example.

The verifier computes:

\\[ c = H(d_3) \\]
\\[ cd = H(c \| d) \\]
\\[ \text{root} = H(ab \| cd) \\]

If the computed root equals the published root, the entry is verified. The proof had \\(\log_2 n\\) hashes — for \\(n = 2^{30}\\) (a billion records), 30 hashes plus the leaf, around 1 KB.

The forge-resistance comes from the second-preimage resistance of the hash. To make the verifier accept a different value at position 3, an attacker would need to find some \\(d_3'\\) and possibly different sibling hashes such that the recomputed root matches. Each step requires a hash collision; the whole forgery requires breaking the hash function.

## Why this scales

Storage at the verifier: 32 bytes (the root).
Proof size per query: \\(O(\log n)\\) hashes.
Verification time per query: \\(O(\log n)\\) hash evaluations.

Compared to alternatives:
- Sign every record individually: \\(O(n)\\) signatures stored.
- Sign a manifest of all hashes: \\(O(n)\\) per query (or full download).
- A Merkle tree pushes verification cost to logarithmic in the dataset size, with constant overhead at the publisher.

For datasets that change, you can also do *Merkle proofs of update*: prove that a new tree differs from the old tree only in a specified set of leaves, in roughly the same logarithmic cost.

## Why git is a Merkle tree

A git repository is a Merkle tree at every level. Each file blob's identity is its content hash. Each tree (directory) is a list of (filename, type, hash); the tree's identity is the hash of that list. Each commit names a root tree, plus parent commits, plus metadata; the commit's identity is the hash of all that.

Two consequences fall out, not as designed features but as byproducts of the construction:

- *Integrity*. Every byte in a git repo is identified by a chain of hashes back to a commit. Corrupt one bit and every hash from that bit up is wrong, including the commit ID.
- *Deduplication*. Files with identical content share the same hash, and therefore the same blob; same for directories. A repository with thousands of identical copies of a file stores it once.

`git fetch` pulls only the objects you do not have, identified by hash, with no central state needed beyond the commit IDs at each end.

## Why blockchains are Merkle trees

Bitcoin, Ethereum, and every other blockchain put every transaction in a Merkle tree per block, and store only the root in the block header. A light client that wants to verify "did transaction \\(T\\) happen in block \\(B\\)" downloads the block header (containing the root), the transaction \\(T\\), and a Merkle proof. That is hundreds of bytes. Without Merkle trees, the client would need the full block.

Ethereum extends this: the entire account-state tree is a Merkle tree (a Merkle Patricia trie, with branching factor 16 and key-prefix structure). The root of this tree, called the state root, is committed in every block header. A few dozen bytes commit to the state of every account on the chain. Light clients verify state without storing it.

## Sparse Merkle trees and Merkle Patricia tries

A standard Merkle tree assumes a list of leaves indexed by position. For key-value data — "what is the value at key \\(k\\)?" — you want indexing by key.

A *sparse Merkle tree* has \\(2^{256}\\) leaves, almost all of them empty. Empty subtrees have known constant hashes (they collapse: the hash of two empty subtrees is the same constant), so most of the tree is implicit. Only a few non-empty paths are stored. Proofs of non-membership ("this key is empty") are well-defined and the same length as proofs of membership.

A *Merkle Patricia trie* is a more sophisticated variant: it compresses long runs of single-child nodes, so the tree branches at most where there are at least two distinct keys. Used for the Ethereum state because state is sparse but billions of keys long.

## Verifiable streaming

A real use that goes beyond static datasets: *certificate transparency*. Every TLS certificate issued by a participating CA is appended to a public Merkle log. Any observer can:

- Get a *signed tree head* — a Merkle root signed by the log operator — periodically.
- Verify that a certificate is included in the log (Merkle inclusion proof).
- Verify that the log is consistent over time, i.e., the root at time \\(t\\) is a prefix of the root at time \\(t' > t\\) (Merkle consistency proof — uses an extension property of Merkle trees that proves a smaller tree is a prefix of a bigger one).

Operators of certificate-transparency logs cannot retroactively delete or alter entries without producing inconsistent root signatures, which other observers would catch. The system has caught real CA misissuances in the wild.

## The wonder

A 32-byte root certifies the contents of an arbitrarily large dataset. To verify any specific entry, a logarithmic-size proof suffices. To detect any tampering, you only have to know the root. To verify consistency over time, you exchange logarithmic-size proofs.

The construction is two lines of pseudocode (`H(left || right)`, recursively). It is the entire content of a fact that the engineering world spent decades not having. Distributed version control, blockchains, certificate transparency, content-addressable storage, BitTorrent piece verification, IPFS — none of these systems would work without it. They look, today, like obvious applications. They were not obvious before Merkle wrote the construction down.

## Where to go deeper

- Ralph Merkle, *Secrecy, Authentication, and Public Key Systems*, Stanford PhD thesis, 1979. The original.
- Laurie, *Certificate Transparency*, ACM Queue 2014. The systems engineering of the largest production Merkle-log deployment.
