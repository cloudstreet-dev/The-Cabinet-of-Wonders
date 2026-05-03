# Zero-knowledge proofs

You can prove to someone, with mathematical certainty, that you know the solution to a problem — without revealing anything about the solution. Not a hint. Not a partial bit. Whoever you are convincing learns one thing only: that you know the answer. After the proof ends, the verifier could not show your proof to a third party as evidence; they cannot reconstruct it.

Goldwasser, Micali, and Rackoff defined this in 1985 and proved it was achievable. It violates the intuition that proving you know X requires letting the verifier see X. The whole edifice of modern privacy-preserving computation — zk-SNARKs, anonymous credentials, blockchain rollups, end-to-end-encrypted login — is downstream of this idea.

## The three properties

A zero-knowledge proof is an interactive protocol between a prover P and a verifier V, parameterized by a statement \\(x\\) and (for P) a witness \\(w\\). It must satisfy:

**Completeness.** If the statement is true and P knows a valid witness, V accepts with probability 1 (or close to it).

**Soundness.** If the statement is false, no prover, no matter how computationally powerful, can make V accept except with negligible probability.

**Zero-knowledge.** Whatever V learns from a proof of a true statement, V could have learned by simulation alone — without P's participation. Formally, there exists an efficient simulator that, given only the statement, produces a transcript indistinguishable from a real proof transcript. So the transcript carries no information beyond the truth of the statement.

The third property is the strange one. Completeness and soundness are routine. Zero-knowledge demands that the protocol leak nothing — but the protocol obviously must convey something to convince the verifier. The trick is that what it conveys is the *truth* of the statement, not the witness, and "truth of the statement" is something the simulator can fake.

## The cave: the canonical illustration

Imagine a cave with two passages forking from an entrance. They reconnect at a far end via a door with a magic spell that only Peggy knows. Victor wants to verify Peggy knows the spell, without learning it.

```
              door (opens with spell)
                  |
        +---------+---------+
        |                   |
       passage A          passage B
        |                   |
        +---------+---------+
                  |
                entrance
```

Protocol:
1. Peggy enters the cave, picks one passage at random, walks to the door.
2. Victor stays at the entrance, then walks to the fork.
3. Victor shouts which passage he wants Peggy to come out of: A or B.
4. If Peggy is already in the demanded passage, she walks back. If she is in the other one, she opens the door with the spell and emerges from the demanded passage.

If Peggy knows the spell, she can always satisfy the request. If she does not, she has to guess in advance which passage Victor will choose; she succeeds with probability 1/2 per round. Repeat \\(k\\) rounds: cheating prover succeeds with probability \\(2^{-k}\\), negligible.

Yet Victor learns nothing about the spell. He could simulate the entire transcript himself: for each round, pick A or B at random, generate a "video" of Peggy emerging from that passage. Without involvement from any spell-knower, his fake transcripts are indistinguishable from real ones (assuming he commits to A or B in advance, as the protocol requires). A real transcript from Peggy and a faked transcript from Victor's imagination look the same.

That last fact is exactly the zero-knowledge property: no information beyond "Peggy knows the spell" is being communicated, because Victor can manufacture indistinguishable evidence on his own.

## Schnorr's protocol: ZK proof of knowledge of discrete log

Concrete example. Public: a group \\(G\\) of prime order \\(q\\) with generator \\(g\\), and an element \\(h = g^x\\). Peggy wants to prove she knows \\(x\\).

1. Peggy picks random \\(r \in \mathbb{Z}_q\\), computes commitment \\(t = g^r\\), sends \\(t\\) to Victor.
2. Victor sends a random challenge \\(c \in \mathbb{Z}_q\\).
3. Peggy computes response \\(s = r + cx \mod q\\), sends \\(s\\).
4. Victor accepts iff \\(g^s = t \cdot h^c\\).

Verification: \\(g^s = g^{r + cx} = g^r \cdot g^{cx} = t \cdot h^c\\). ✓

Soundness: a cheating prover who does not know \\(x\\) must commit to \\(t\\) before seeing \\(c\\). For each choice of \\(t\\), at most one \\(c\\) admits a valid response (the prover would have to know \\(x\\) to compute responses for two different \\(c\\)s, since two valid \\((c_1, s_1)\\) and \\((c_2, s_2)\\) give \\(x = (s_1 - s_2)/(c_1 - c_2)\\)). So cheating succeeds with probability at most \\(1/q\\), negligible.

Zero-knowledge: a simulator can produce a transcript \\((t, c, s)\\) by picking \\(c\\) and \\(s\\) at random and setting \\(t = g^s / h^c\\). The simulated transcript is identically distributed to a real one (both have uniform \\(c\\), \\(s\\), and the corresponding \\(t\\)). Verification succeeds. The simulator did not need \\(x\\).

Three messages, modular arithmetic. Universally implemented, including inside Schnorr signatures (Bitcoin, since 2021), Ed25519 (every modern SSH key), and innumerable identification schemes.

## Fiat–Shamir: making it non-interactive

Schnorr's protocol is interactive: Victor's challenge \\(c\\) is sent in real time. The Fiat–Shamir heuristic replaces \\(c\\) with \\(\text{Hash}(t \| \text{statement})\\). The hash function acts as a "random oracle": its output is unpredictable to the prover until they have committed to \\(t\\), so the soundness analysis carries over (with the random-oracle assumption). The protocol becomes non-interactive: the prover sends \\((t, s)\\) once, the verifier checks.

This is the dominant technique for converting interactive ZK protocols into digital-signature schemes. Schnorr signatures, EdDSA, almost all post-quantum signature candidates rely on Fiat–Shamir.

## SNARKs: ZK for arbitrary computation

The big modern construction is the Succinct Non-interactive ARgument of Knowledge, or zk-SNARK. It compiles an *arbitrary* computation — any program — into a system where the prover can produce a constant-size proof that they ran the program correctly on some private input, and the verifier can check the proof in milliseconds.

The pipeline:
1. Compile the computation to an arithmetic circuit over a finite field.
2. Express correctness as a polynomial identity (R1CS, or PLONKish constraint systems).
3. Convert that into a "polynomial-IOP" — an interactive protocol where the prover commits to polynomials and the verifier checks evaluations at random points (Schwartz–Zippel ensures correctness).
4. Use polynomial commitments (KZG, FRI, Bulletproofs) so the verifier can check evaluations without seeing the whole polynomial.
5. Apply Fiat–Shamir to remove interaction.

The result: the prover does work proportional to the circuit size. The verifier does logarithmic or constant work. The proof is hundreds of bytes to a few kilobytes.

This is the engine behind Zcash (private transactions: prove a transaction is valid without revealing amounts or parties), zk-rollups on Ethereum (prove that a batch of thousands of transactions executed correctly, post just the proof to L1), and a steady drumbeat of new privacy-preserving applications.

## What zero-knowledge gets you

- *Authentication without password leakage.* Prove you know the password without sending it.
- *Anonymous credentials.* Prove you are over 18, or a citizen, without revealing your name or birth date.
- *Verified computation.* Hand a worker your data, get back a proof that they computed correctly, without trusting their hardware.
- *Privacy-preserving cryptocurrency.* Prove a transaction balances without revealing senders, receivers, or amounts.
- *Constant-size proofs of long execution.* Prove a virtual machine executed a million instructions, in a 200-byte proof.

## The wonder

Zero-knowledge proofs answer a question that, before 1985, did not look like a coherent question: "can you prove you know something without revealing what you know?" The naive answer is no — a proof must transmit *something*, and a sufficiently clever verifier can reconstruct from anything you transmit. The right answer is yes, because what the verifier can extract from the protocol is bounded above by what they could extract by themselves with no protocol at all. The simulator is the witness to this. If the verifier could fake a transcript on their own, they cannot blame any leakage on the prover. So the prover leaks nothing.

The construction is a rigorous mathematical formalization of "convincing without telling." It is one of those cases where the right definition is itself the entire breakthrough; once you have the simulator-based definition, the constructions follow.

## Where to go deeper

- Goldwasser, Micali, Rackoff, *The Knowledge Complexity of Interactive Proof-Systems*, STOC 1985. The defining paper.
- Justin Thaler, *Proofs, Arguments, and Zero-Knowledge* (free online). Modern textbook covering the SNARK landscape.
