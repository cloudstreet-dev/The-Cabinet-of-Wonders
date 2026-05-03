# Verifiable random functions

You publish a public key. Someone hands you an input. You produce two things: a deterministic random-looking output, and a proof. Anyone with your public key can use the proof to verify the output was computed correctly. Anyone *without* your secret key cannot guess the output for an unseen input — to them, your function looks like a random oracle, even though it is deterministic and verifiable.

So you have a function whose output is unpredictable to outsiders, predictable to you, and *proved* correct after the fact. This combination — which sounds like it might be impossible — is the engine running underneath modern proof-of-stake consensus and other randomness-with-accountability protocols.

## The contract

A verifiable random function gives three operations:

- **Keygen** → (sk, pk).
- **Eval(sk, x)** → (y, π). \\(y\\) is the output; \\(\pi\\) is the proof.
- **Verify(pk, x, y, π)** → boolean.

It must satisfy:

**Uniqueness.** For any \\((pk, x)\\), there is exactly one \\(y\\) such that some \\(\pi\\) makes Verify accept. The function is well-defined as a public mathematical object, regardless of what the secret-key holder chooses to claim.

**Pseudorandomness.** Without the secret key, given any number of \\((x_i, y_i, \pi_i)\\) pairs for chosen \\(x_i\\), the output \\(y\\) on a fresh \\(x\\) is computationally indistinguishable from uniform.

**Provability.** With the secret key, the output and proof are efficiently computable, and Verify always accepts honest proofs.

## A clean construction (BLS-VRF)

Public parameters: a group \\(G\\) with a pairing \\(e: G \times G \to G_T\\), generator \\(g\\), and a hash-to-group function \\(H: \{0,1\}^* \to G\\).

- **Keygen**: pick \\(sk \in \mathbb{Z}_q\\) at random, compute \\(pk = g^{sk}\\).
- **Eval(sk, x)**: compute \\(\pi = H(x)^{sk}\\) and \\(y = \text{Hash}(\pi)\\) (where the second hash is to the desired output space). Output \\((y, \pi)\\).
- **Verify(pk, x, y, π)**: check that \\(e(\pi, g) = e(H(x), pk)\\) and that \\(y = \text{Hash}(\pi)\\).

Why it works:

- *Uniqueness*: \\(\pi\\) is the unique BLS signature on \\(x\\) under \\(sk\\). The pairing equation \\(e(\pi, g) = e(H(x), pk)\\) holds iff \\(\pi = H(x)^{sk}\\). And \\(y\\) is determined by \\(\pi\\). So \\(y\\) is unique.
- *Pseudorandomness*: under the bilinear Diffie–Hellman assumption, \\(H(x)^{sk}\\) is computationally indistinguishable from random for unseen \\(x\\). Hashing makes the output statistically uniform.
- *Provability*: secret-key holder computes \\(H(x)^{sk}\\) directly.

Three lines. Beautiful, compact, deployable.

## Why this is not just a signature

A signature also gives uniqueness and provability (verifiable, computed from a secret key). What signatures do *not* give is pseudorandomness of the signature itself. A standard ECDSA signature is randomized — different signatures of the same message are different, and not pseudorandom in any useful sense.

A VRF needs the output to be a deterministic, pseudorandom function of the input. Deterministic, so it is well-defined; pseudorandom, so others cannot predict it. Most signatures fail this. BLS signatures happen to be deterministic and pseudorandom (under appropriate assumptions), so BLS-VRF is just BLS reinterpreted as a VRF. Schnorr signatures, with care, can be made into a VRF too.

## What VRFs unlock: leader election in PoS

The killer application is leader election in proof-of-stake consensus protocols (Algorand, Cardano, Ethereum's beacon chain, and many others).

Each block, the protocol needs to pick a leader from among the validators. The pick should be:

- **Random** — no one can predict who will be leader far in advance, so attackers cannot target the leader for DoS.
- **Verifiable** — once a leader claims they were elected, anyone can check.
- **Decentralized** — no oracle or beacon, no group VDF computation, no expensive MPC for each block.

VRF gives you all three. Each validator computes \\(y_i = \text{VRF}(sk_i, \text{seed}\| \text{slot})\\). They publish \\(y_i\\) along with proof \\(\pi_i\\). The validator with the smallest \\(y_i\\) (or some weighted variant) is the leader for the slot. Until they publish, no one knows who has the smallest output. After they publish, everyone verifies the proof and the order.

This is unforgeable (validators cannot pick a different output for themselves), unpredictable (no one can guess outputs of other validators' VRFs), self-evident (no global computation required). The protocol scales to thousands of validators with minimal communication.

## Why this is wonder, not just engineering

Strip the technical content away. You have a function. The function is a deterministic mathematical object — for each input, there is a unique correct output, baked into the public parameters. But: to compute the output, you need a secret. To verify the output is the correct one, you do not need the secret. To predict the output without computing it, you would need to break a hard cryptographic problem.

That is a strange object. It behaves like a random oracle (a notional black box that returns a fresh random value for each query, no shortcut) — but to one specific party, the secret-key holder, it is computable, deterministic, and they can prove their answers are right.

You can think of it as a private-public split on randomness itself: the holder of the secret key sees the function as a deterministic mapping; everyone else sees it as a random oracle. The protocol can use the determinism (everyone agrees the leader is uniquely defined) and the randomness (no one can predict who will be leader) at the same time, because the two views coexist coherently.

## Where to go deeper

- Micali, Rabin, Vadhan, *Verifiable Random Functions*, FOCS 1999. The defining paper.
- Goldberg, Naor, Papadopoulos, Reyzin, *Verifiable Random Functions (VRFs)*, RFC 9381 (2023). The IETF standard with construction details and security analysis.
