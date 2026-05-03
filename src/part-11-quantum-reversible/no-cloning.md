# The no-cloning theorem

There is no machine, even in principle, that can take an unknown quantum state and produce two identical copies of it. The operation that maps \\(|\psi\rangle \otimes |0\rangle\\) to \\(|\psi\rangle \otimes |\psi\rangle\\) for arbitrary \\(|\psi\rangle\\) is *not unitary*, and quantum mechanics permits only unitary operations. So the cloning function does not exist as a quantum operation.

The theorem is one line of algebra. It was proved by Wootters and Zurek (1982) and independently by Dieks (1982). It is the cornerstone of quantum cryptography (BB84's security), the explanation for why quantum information has a fundamentally different character than classical information, and a non-negotiable design constraint on every quantum protocol.

## The proof

Suppose, for contradiction, that there exists a unitary \\(U\\) and a "blank" state \\(|0\rangle\\) such that for *every* \\(|\psi\rangle\\),

\\[ U(|\psi\rangle \otimes |0\rangle) = |\psi\rangle \otimes |\psi\rangle \\]

Apply this to two different states \\(|\psi\rangle\\) and \\(|\phi\rangle\\):

\\[ U(|\psi\rangle \otimes |0\rangle) = |\psi\rangle \otimes |\psi\rangle \\]
\\[ U(|\phi\rangle \otimes |0\rangle) = |\phi\rangle \otimes |\phi\rangle \\]

Take the inner product of the two equations. Since \\(U\\) is unitary, it preserves inner products:

\\[ \langle \psi | \phi \rangle \cdot \langle 0 | 0 \rangle = (\langle \psi | \phi \rangle)^2 \\]

\\[ \langle \psi | \phi \rangle = (\langle \psi | \phi \rangle)^2 \\]

So either \\(\langle \psi | \phi \rangle = 0\\) (orthogonal) or \\(\langle \psi | \phi \rangle = 1\\) (identical). The cloning operation can only work for state pairs that are either orthogonal or identical — not for arbitrary pairs. There is no universal cloner. \\(\blacksquare\\)

## What this rules out

You cannot:
- Make a copy of an unknown qubit.
- Run a quantum subroutine multiple times on the same input by cloning the input.
- "Save" a quantum state for later use without measurement (you can transfer it to a quantum memory, but not duplicate it).
- Eavesdrop on a quantum channel without disturbing it (the cornerstone of QKD security).

You *can*:
- Copy a known classical state (since you can prepare arbitrary copies from the description).
- Copy *orthogonal* states (since the constraint is only on arbitrary pairs).
- Approximately clone — the optimal approximate cloner has fidelity \\(5/6\\) for symmetric universal cloning of qubits; this is the best possible without violating no-cloning. (Buzek-Hillery 1996.)
- "Move" a state via teleportation, which destroys the original.
- Distribute entanglement, which creates correlated quantum systems (not copies in the cloning sense).

## Why it underlies QKD

In BB84 (a separate entry in this part), Alice sends Bob qubits in randomly-chosen bases. Eve cannot intercept-and-resend without measuring. Measuring requires choosing a basis. If Eve guesses wrong, her measurement disturbs the state, and Bob's measurement (even when in Alice's correct basis) yields a mismatch with probability 1/2.

If cloning were possible, Eve could clone each photon, send the original on to Bob, and measure her own copy in any basis (or wait until Alice and Bob announce bases, then measure in the announced basis). She would learn the bit without disturbing the original. QKD would be insecure.

No-cloning prevents this. Eve must measure-and-disturb, and the disturbance is detectable. QKD's security is a direct corollary.

## Why it is *exactly* what is needed

Many quantum mechanics constraints feel like incidental restrictions ("you can't do measurement \\(X\\) without disturbance"). No-cloning is more structural: it follows directly from unitarity, the most basic feature of quantum dynamics.

Unitarity preserves inner products. The cloning operation would *square* inner products. The two are incompatible. There is no clever hardware trick or alternative formulation; the impossibility is enforced by quantum mechanics's most fundamental property.

This makes no-cloning rare among quantum-mechanics impossibilities: it is not "we don't know how" but "the rules forbid it for first-principles reasons."

## What it permits

No-cloning forbids exact universal cloning. It does not forbid:

**Approximate cloning**: produce two states each of fidelity \\(F < 1\\) with the input. Optimal universal cloners (no knowledge of input) achieve \\(F = 5/6 \approx 0.833\\) for two-qubit cloning. State-dependent cloners (knowing the input is one of a fixed set) can do better.

**Cloning of orthogonal states**: if the input is known to be in some fixed orthogonal basis, cloning is fine. This is just classical copying in disguise.

**Cloning of partial information**: you can copy *some* properties of a state — its expectation values, its measurement statistics — at the cost of destroying the original.

**Cloning into entangled outputs**: the optimal cloner produces *entangled* outputs that are individually approximate clones; their joint correlation carries the missing information.

## The no-deleting theorem

The dual: there is no operation that takes \\(|\psi\rangle \otimes |\psi\rangle\\) to \\(|\psi\rangle \otimes |0\rangle\\) for arbitrary \\(|\psi\rangle\\). Quantum information cannot be deleted any more than it can be cloned. This is the *no-deleting theorem* (Pati and Braunstein 2000), with a similar one-line proof.

Together, no-cloning and no-deleting say that quantum information is *conserved*: it cannot be duplicated, it cannot be destroyed. In a quantum protocol, the total amount of quantum information is preserved end-to-end.

## What it implies for engineering

Every quantum protocol must respect no-cloning. So:

- **Quantum error correction** cannot work by storing the same information in multiple places. Instead, it stores logical information in *entangled* combinations of physical qubits. The Shor code and Steane code embed one logical qubit in 9 or 7 physical qubits, with redundancy that protects against errors without violating no-cloning.
- **Quantum repeaters** for long-distance communication cannot just amplify the signal (which would clone). They must use entanglement swapping — successive teleportations along intermediate nodes — to extend reach.
- **Quantum memory** stores quantum states intact, but cannot duplicate them.
- **Quantum random-access memory (qRAM)** is a research challenge precisely because random access without disturbance and without cloning is non-trivial.

## The wonder

Classical information is freely copyable, infinitely. Bits flow from disks to networks to RAM to caches, with copies in many places, freely. Quantum information is fundamentally different: it can be moved, but never duplicated. Each qubit's worth of information has a location at any moment, and only one location.

This single fact reshapes every quantum protocol. Cryptography becomes inherently secure (eavesdropping is observable). Networking becomes harder (no signal regeneration by amplification). Programming becomes more delicate (no reusing inputs). Storage becomes harder (no backup by copying).

Yet the same restriction is *what makes quantum information distinct*. Classical information is everywhere copyable, and as a result has no special structure beyond what classical Shannon theory describes. Quantum information is rare and fragile, and as a result encodes correlations and structures that classical systems literally cannot have. The restriction is also the gift.

The wonder is in the proof: one line of inner-product algebra, applied to the demand that a copy machine work for arbitrary unknown states. Unitary preserves inner products; copying squares them; therefore copying isn't unitary; therefore no copy machine exists. A foundational impossibility result, established with high-school linear algebra.

## Where to go deeper

- Wootters and Zurek, *A single quantum cannot be cloned*, Nature 1982. The original.
- Nielsen and Chuang, *Quantum Computation and Quantum Information*, Section 12.1. Modern textbook.
