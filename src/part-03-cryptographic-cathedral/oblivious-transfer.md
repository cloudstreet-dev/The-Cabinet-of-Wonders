# Oblivious transfer

You have two sealed envelopes. I want exactly one of them, and I want to choose which without telling you. After our exchange I have my chosen envelope's contents, and you have no idea which one I took. Also: I learn nothing about the envelope I did not pick.

This is oblivious transfer. It looks structurally impossible, because for me to receive an envelope you have to send it, but you have to send it without knowing which I want. Yet it is achievable, with two messages and a single Diffie–Hellman-like operation.

It is the foundation under garbled-circuit MPC, private information retrieval, and several other constructions. Kilian (1988) showed something stronger: oblivious transfer is computationally complete for secure computation. Given OT, you can build any MPC protocol.

## The 1-out-of-2 specification

Sender has two messages \\(m_0, m_1\\). Receiver has a choice bit \\(b \in \{0, 1\}\\). After the protocol:

- Receiver learns \\(m_b\\).
- Receiver learns nothing about \\(m_{1-b}\\).
- Sender learns nothing about \\(b\\).

That is the contract. The construction has to satisfy it against active adversaries.

## The Bellare-Micali / Naor-Pinkas construction

Public parameters: a group \\(G\\) of prime order \\(q\\), generator \\(g\\), where Computational Diffie–Hellman is hard.

The protocol:

1. **Sender** picks a random \\(c\\) and sends it. (\\(c\\) is a group element, fixed for this exchange.)
2. **Receiver** picks random \\(k \in \mathbb{Z}_q\\). If \\(b = 0\\), set \\(\text{PK}_0 = g^k\\), \\(\text{PK}_1 = c \cdot \text{PK}_0^{-1}\\). If \\(b = 1\\), set \\(\text{PK}_1 = g^k\\), \\(\text{PK}_0 = c \cdot \text{PK}_1^{-1}\\). Send \\((\text{PK}_0, \text{PK}_1)\\) to sender.

   Crucial property: \\(\text{PK}_0 \cdot \text{PK}_1 = c\\) always. The receiver knows the discrete log of one of \\((\text{PK}_0, \text{PK}_1)\\) — namely \\(k\\) — and not of the other (computing the other would require finding the discrete log of \\(c \cdot g^{-k}\\), which is hard). And from the sender's perspective, both pairs (with \\(b = 0\\) and \\(b = 1\\)) are uniformly distributed and indistinguishable.

3. **Sender** picks random \\(r_0, r_1\\) and sends back \\((g^{r_0}, m_0 \oplus H(\text{PK}_0^{r_0}))\\) and \\((g^{r_1}, m_1 \oplus H(\text{PK}_1^{r_1}))\\), where \\(H\\) is a hash function used as a key-derivation function.

4. **Receiver**, who knows the discrete log \\(k\\) of \\(\text{PK}_b\\), computes \\(\text{PK}_b^{r_b} = (g^{r_b})^k\\), hashes it, XORs with the relevant ciphertext, and recovers \\(m_b\\). For the *other* message, the receiver does not know the discrete log of \\(\text{PK}_{1-b}\\), so cannot compute \\(\text{PK}_{1-b}^{r_{1-b}}\\) from \\(g^{r_{1-b}}\\), so cannot decrypt \\(m_{1-b}\\).

That is one-out-of-two oblivious transfer in three messages and a few exponentiations. The sender does not know \\(b\\) (the message in step 2 is uniform whichever choice). The receiver cannot recover \\(m_{1-b}\\) (would require solving CDH).

## OT extension

OT is expensive — public-key operations are slow. For applications like garbled-circuit MPC, you might need millions of OTs per protocol run.

Ishai-Kilian-Nissim-Petrank (2003) showed how to "extend" OT: do a small number \\(\kappa\\) (say, 128) of "base OTs" with public-key operations, then derive any number \\(N\\) of subsequent OTs using only symmetric-key operations (hash functions, AES). The cost per OT after the base setup is essentially the cost of a hash plus communication of two ciphertexts.

This is what makes garbled-circuit MPC fast in practice. The base OTs cost is fixed; the per-gate cost scales linearly in the number of gates with a small constant. Without OT extension, MPC of any nontrivial circuit would be untenable.

## Oblivious transfer as the cryptographic atom

Kilian's 1988 result: any two-party functionality can be securely computed given black-box access to oblivious transfer. OT is a *cryptographic complete primitive*. Anything cryptography can do (in two-party world, in the appropriate model) can be reduced to OT plus elementary computation.

This is one of those results that elevates the wonder. OT looks like a narrow specialty primitive — "I want one of two messages." But the operation has the right structure to encode arbitrary boolean computation, because each gate of a circuit can be implemented by a sufficiently elaborate use of OT (in particular, garbled-circuit-style or its successors). Once you have OT, you have all of cryptography (in the right model).

The reverse is also true: OT cannot be built from one-way functions alone in the standard model. It seems to require something with more structure — public-key cryptography or a noisy channel. So OT is, in some sense, a marker of a cryptographic regime.

## A few uses

- **Garbled circuits**: receiver learns labels for their input wires without sender learning which (one OT per input bit).
- **Private information retrieval**: client wants record \\(i\\) from a database without server learning \\(i\\). Implementable from OT plus communication tricks.
- **Private set intersection**: two parties find shared elements without revealing the rest.
- **MPC compilers**: more general protocol stacks built on OT-extension primitives, used in production secure-aggregation systems.

## Why it should not work

Run the intuition: I want one of two things, you have to give it to me, but without knowing which. So you have to send both — but if you send both, I get both. Resolution: you do not send the messages directly; you send each *encrypted* under a key that only the receiver-of-that-message can derive. I can derive the key for one of them (the one I chose) but not the other. So you send both ciphertexts. Only one decrypts for me. From your side, you cannot tell from the keys-I-could-derive which one I am set up to receive (the choice information was hidden by the algebraic structure of the public keys).

The argument has three moving parts: the public keys disguise the receiver's choice; the encryption keys are non-extractable from the public keys without a secret the receiver controls for one of them; the sender's randomness ensures the unrelated ciphertext is uniformly random as far as the receiver can tell. Each is a few lines of algebra. Together they do something that, on paraphrase, sounds like a contradiction.

## Where to go deeper

- Naor and Pinkas, *Efficient Oblivious Transfer Protocols*, SODA 2001. The protocol described above, with security proofs.
- Ishai, Kilian, Nissim, Petrank, *Extending Oblivious Transfers Efficiently*, CRYPTO 2003. The OT-extension construction.
