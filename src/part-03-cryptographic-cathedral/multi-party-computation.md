# Multi-party computation

A group of people, each holding a private input, want to compute a function of all the inputs and learn the output — without anyone learning anyone else's input. They might not trust each other. Some of them might be lying or actively trying to cheat. They run a protocol, and at the end everyone has the answer, and that is *all* anyone has learned. As if there were a trusted third party who took everyone's secrets, computed the answer, and announced it — except there is no trusted third party.

Yao introduced the concept in 1982 with the millionaires' problem: two millionaires want to know who is richer without revealing how rich. The general solution exists. It runs in the real world, today, on real workloads.

## The model

\\(n\\) parties \\(P_1, \dots, P_n\\) each hold a private input \\(x_i\\). They want to compute \\(f(x_1, \dots, x_n)\\) for some agreed function \\(f\\). At the end of the protocol:

- Every honest party learns \\(f(x_1, \dots, x_n)\\).
- No coalition of corrupted parties learns anything about the honest parties' inputs beyond what \\(f\\) and their own inputs already imply.

The corruption model matters. *Semi-honest* (passive) corruption: corrupted parties follow the protocol but record everything they see. *Malicious* (active) corruption: corrupted parties deviate from the protocol arbitrarily. The latter is much harder to defend against.

Number of corruptions matters too. Honest majority: more than half are honest. Dishonest majority: anyone could be corrupted. Different protocols handle different thresholds.

## Yao's garbled circuits

Two parties, \\(A\\) and \\(B\\), with inputs \\(x_A\\) and \\(x_B\\). Compute \\(f(x_A, x_B)\\).

\\(A\\) takes the boolean circuit for \\(f\\) and *garbles* it. Each wire in the circuit is assigned two random labels (long random bit-strings) — one representing the wire-value 0, one representing the wire-value 1. For each gate, \\(A\\) constructs a garbled truth table: each row of the table is the output label encrypted under the two input labels for that input combination. The rows are randomly permuted so the row order does not leak which combination is which. \\(A\\) sends the garbled circuit to \\(B\\), along with the labels for \\(A\\)'s own input wires.

For \\(B\\)'s input wires, \\(A\\) does not know the value, so \\(A\\) cannot just send labels (each input value has its own label and revealing both would let \\(B\\) cheat). Instead, \\(A\\) and \\(B\\) run an *oblivious transfer* protocol: for each input bit \\(b_i\\) of \\(B\\), \\(B\\) selects the label corresponding to \\(b_i\\) without \\(A\\) learning which one. (See *Oblivious transfer* in this part.)

\\(B\\) now has labels for every input wire and the garbled circuit. \\(B\\) evaluates: for each gate, the labels on the input wires are exactly the keys needed to decrypt one row of the garbled truth table — the one corresponding to the actual input combination. The decrypted value is the label for the output wire. The other rows look like noise. \\(B\\) propagates labels through the circuit, ending at output wires.

\\(A\\) reveals which output label means 0 and which means 1, and now \\(B\\) knows the output. \\(B\\) tells \\(A\\), or both keep it private, depending on the function.

The construction makes the entire computation depend on labels that \\(B\\) cannot relate to plaintext values. \\(B\\) sees one label per wire; the *other* label, which would reveal the bit value, is never sent. \\(A\\) sees the structure but not \\(B\\)'s inputs. The trusted-third-party model is simulated by encryption keys neither side can fully see.

A circuit with \\(N\\) gates needs \\(O(N)\\) ciphertexts. Modern implementations (with optimizations like *half gates* and *free XOR*) need a few hundred bits per gate, encrypt at AES-NI speeds, and evaluate AES on the millions of gates needed for typical workloads in a fraction of a second.

## GMW: secret-sharing-based MPC

Goldreich-Micali-Wigderson (1987) generalized Yao to arbitrary numbers of parties using a different mechanism: secret sharing.

A value \\(x \in \mathbb{F}_2\\) is shared among \\(n\\) parties as \\(x_1, x_2, \dots, x_n\\) where \\(x = x_1 \oplus x_2 \oplus \cdots \oplus x_n\\). Each party gets one share; shares are uniformly random subject to that constraint, so any subset of fewer than \\(n\\) shares reveals nothing.

To compute on shared values:
- **Addition (XOR)**: each party XORs their shares of the inputs. Local computation, no communication.
- **Multiplication (AND)**: requires interaction. Use a "Beaver triple": three pre-shared values \\(([a], [b], [c])\\) with \\(c = ab\\), where the brackets mean shared. Parties locally compute shares of \\(d = x - a\\) and \\(e = y - b\\), open them, then compute \\([xy] = de + d[b] + e[a] + [c]\\). One round of communication per multiplication.

The hard part is generating the multiplication triples. Done in an offline phase, before the actual inputs are known, using oblivious transfer or homomorphic encryption. Once the offline phase is finished, online execution is fast and cheap.

For Shamir-secret-shared MPC (BGW 1988), shares are evaluations of polynomials and the threshold of corrupted parties is bounded by the polynomial degree, but the operations are similar. Addition is local; multiplication requires interaction (and degree reduction).

## What can MPC compute today

In production, on real users:
- **Boston Women's Workforce Council**: 100+ companies' encrypted salary data, MPC computed gender pay gap statistics across the city. Companies never saw each other's data; the public got the aggregate.
- **Estonia's tax fraud detection**: MPC across multiple government agencies' siloed data.
- **Cryptocurrency cold-wallet signing**: threshold ECDSA across multiple devices, no single device has the private key but a quorum can sign. Used by major custodians.
- **Privacy-preserving analytics**: e.g., Apple's iOS analytics use MPC-derived mechanisms to compute population statistics without seeing per-device data.

Performance has been the bottleneck. A general MPC of a billion-gate circuit between two parties might take minutes on commodity hardware; for 10+ parties, hours to days. For specific functions optimized at the protocol level (matrix products, neural-network inference, simple statistical aggregations), throughput is high enough for real applications.

## Threshold cryptography as MPC

Many specialized MPC protocols are bundled under "threshold cryptography": a cryptographic operation (signature, decryption, key generation) is split among \\(n\\) parties such that any \\(t\\) of them can perform the operation, but fewer than \\(t\\) cannot. This is just MPC for the specific function "compute the signature/decryption."

Threshold ECDSA, threshold BLS, threshold RSA — all are now feasible. They underpin secure custody systems and are starting to show up in distributed key management for institutional crypto holdings.

## The Goldreich-Micali-Wigderson theorem

The 1987 result is striking: *any* polynomial-time computable function can be evaluated under secure multi-party computation, against any number of semi-honest adversaries (as long as fewer than \\(n\\) — i.e., at least one honest party). And against malicious adversaries, the same is true under cryptographic assumptions, with bounded round complexity.

In other words: secure multi-party computation is, in principle, *general*. There is no function the protocol designer is forced to leave unprotected. Whatever you can compute classically with a trusted third party, you can compute with no trusted third party at all — paying only a polynomial overhead.

## The wonder

The intuition that the trusted third party is necessary turns out to be wrong. You can simulate one with mathematics. Encrypt the inputs in a way that everyone can verify the computation is correct, but no one can read the inputs. Then everyone computes on encrypted data, gets encrypted intermediate values, and at the end decrypts only the answer. The trick is to do this without a higher-trust setup; the participants together generate whatever cryptographic material they need, and any subset within the corruption threshold cannot break it.

For most of human history, "we want to combine information without leaking individual contributions" required a referee, a notary, an auctioneer, or some other trusted figure. Multi-party computation says: not anymore. The math can play the referee.

## Where to go deeper

- Yao, *Protocols for Secure Computations*, FOCS 1982, and *How to Generate and Exchange Secrets*, FOCS 1986. Garbled circuits.
- Goldreich, Micali, Wigderson, *How to Play Any Mental Game*, STOC 1987. The general theorem.
- Evans, Kolesnikov, Rosulek, *A Pragmatic Introduction to Secure Multi-Party Computation* (2018, free online). Modern protocols and engineering.
