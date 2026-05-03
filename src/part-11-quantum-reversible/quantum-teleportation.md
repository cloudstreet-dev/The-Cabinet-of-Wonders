# Quantum teleportation

You have a qubit in some unknown quantum state \\(|\psi\rangle\\). You want to send it to your friend across the galaxy. Quantum mechanics forbids you from copying it (no-cloning theorem) or measuring it without destroying its information. So you cannot simply describe the state and email it.

But: if you and your friend share an *entangled pair* of qubits in advance, and you send two *classical bits* over a normal communication channel, your friend can reconstruct \\(|\psi\rangle\\) exactly. Their qubit ends up in the state your qubit was in. Yours is now destroyed (the no-cloning is preserved). The state has been "teleported" — disassembled, transmitted as classical bits and a pre-existing entanglement, and reassembled.

This is real. It has been done in laboratories with photons (Bouwmeester et al. 1997, since then routinely), with ions, with atoms, even at intercontinental scales using satellite-based entanglement distribution (Pan group, 2017). It is the foundational primitive of quantum networks.

## The setup

Three qubits:
- \\(A\\): held by Alice, in some unknown state \\(|\psi\rangle = \alpha |0\rangle + \beta |1\rangle\\) (the "payload").
- \\(B, C\\): an entangled pair, prepared together previously. Alice has \\(B\\); Bob (her friend across the galaxy) has \\(C\\). Their joint state is the *Bell state* \\(|\Phi^+\rangle = (|00\rangle + |11\rangle)/\sqrt{2}\\).

So initially:

\\[ |\psi\rangle_A \otimes |\Phi^+\rangle_{BC} = (\alpha |0\rangle + \beta |1\rangle) \otimes (|00\rangle + |11\rangle)/\sqrt{2} \\]

\\[ = (\alpha |000\rangle + \alpha |011\rangle + \beta |100\rangle + \beta |111\rangle)/\sqrt{2} \\]

(Subscripts on kets denote which qubit; first slot \\(A\\), second \\(B\\), third \\(C\\).)

## The protocol

Alice performs a *Bell-basis measurement* on her two qubits \\(A\\) and \\(B\\). The Bell basis is the four entangled states

\\[ |\Phi^\pm\rangle = (|00\rangle \pm |11\rangle)/\sqrt{2} \\]
\\[ |\Psi^\pm\rangle = (|01\rangle \pm |10\rangle)/\sqrt{2} \\]

Alice's measurement projects \\(A, B\\) onto one of these four states with equal probability 1/4. The remaining qubit \\(C\\) (with Bob) is left in some state correlated with what Alice measured.

By writing the initial state in the Bell basis (a few lines of algebra), you find:

\\[ |\Phi^+\rangle_{AB} \otimes (\alpha |0\rangle + \beta |1\rangle)_C \quad \text{if Alice measures } |\Phi^+\rangle \\]
\\[ |\Phi^-\rangle_{AB} \otimes (\alpha |0\rangle - \beta |1\rangle)_C \quad \text{if Alice measures } |\Phi^-\rangle \\]
\\[ |\Psi^+\rangle_{AB} \otimes (\beta |0\rangle + \alpha |1\rangle)_C \quad \text{if Alice measures } |\Psi^+\rangle \\]
\\[ |\Psi^-\rangle_{AB} \otimes (\beta |0\rangle - \alpha |1\rangle)_C \quad \text{if Alice measures } |\Psi^-\rangle \\]

Bob's qubit ends up in one of four states — each related to \\(|\psi\rangle\\) by a known *Pauli operation* (\\(I, Z, X, X Z\\)).

Alice tells Bob which Bell state she measured (two classical bits of information). Bob applies the corresponding Pauli operator to his qubit. The result: his qubit is now in state \\(|\psi\rangle\\). Teleportation complete.

## What got transmitted

Alice's qubit \\(A\\) is destroyed by the Bell-basis measurement. Bob's qubit \\(C\\) is in state \\(|\psi\rangle\\). Two classical bits crossed the channel. The total information cost: 1 qubit teleported, 2 classical bits sent, 1 entangled pair consumed.

The two classical bits cannot, by themselves, encode an arbitrary qubit (qubits live in a continuous state space; two bits is just four discrete values). The bulk of the "information" came from the pre-shared entanglement, plus the consumed input qubit on Alice's side. The classical bits told Bob *how to interpret* his end of the entanglement, given Alice's measurement outcome.

## Why no faster-than-light communication

The instant Alice measures, Bob's qubit collapses to one of four states. Could Bob detect this and infer something faster than light?

No. Without knowing Alice's measurement outcome, Bob's state is *equally likely* to be any of the four corrected versions of \\(|\psi\rangle\\). Averaged over the four possibilities (each with probability 1/4), Bob's qubit is in the *maximally mixed state* — it carries no information at all. Bob has to wait for Alice's classical message (limited by the speed of light) to know which Pauli to apply. Until then, his qubit is useless.

So entanglement provides correlation but not communication. The speed-of-light limit on classical bits is what prevents faster-than-light signaling.

## Why this is profound

Imagine packing the full information of a quantum state — including, in principle, an unbounded number of complex parameters describing superpositions — into two classical bits *plus* a pre-shared entanglement. The classical bits are tiny; the entanglement is finite but pre-positioned. Together they suffice.

The teleportation protocol is the operational meaning of "quantum information is a real resource that can be transmitted." It demonstrates that the substrate of quantum mechanics is sharable, transferable, and recoverable, not stuck inside a single physical system.

## Real-world implementations

**Photons (1997 to present).** Two photons entangled in polarization, distributed to Alice and Bob; Alice has a third photon in some unknown state; Bell-basis measurement (a partial one, since linear optics cannot perform a complete Bell measurement without auxiliary photons, but enough to teleport). Demonstrated over progressively longer distances: meters, kilometers, free-space links, fiber optics.

**Satellite-to-ground (2017).** The Chinese satellite Micius distributed entangled photon pairs from low-Earth orbit to ground stations, used them for teleportation over distances exceeding 1000 km. Bouwmeester's earlier 1997 work showed the principle; Pan's group scaled it up.

**Ions (2004).** Two trapped ions entangled via a laser-mediated interaction; teleportation of an ion's internal state.

**Quantum memories (2010s).** Teleportation between solid-state quantum memories (e.g., NV centers in diamond) over kilometers of fiber.

The technology is mature enough that quantum-network testbeds (the Quantum Internet Alliance, the Boston-Area Quantum Network) are building infrastructure for teleportation as a primitive.

## What it does not do

It does not transmit matter. The qubit's *information* is teleported; Bob's qubit was already there.

It does not exceed the speed of light for signal transmission, by the no-signaling argument.

It does not duplicate the original — Alice's copy is destroyed by the measurement (the no-cloning theorem is preserved).

## The wonder

The state of a qubit — a continuous-parameter object — can be perfectly transmitted using only:
- A pre-shared entangled pair (two qubits worth of resource, prepared in advance).
- Two classical bits sent through any normal communication channel.

The quantum information was never *in motion* in any classical sense. No subatomic particle traveled from Alice to Bob carrying \\(|\psi\rangle\\). The protocol is non-local in a way no classical analog has: the pre-shared entanglement acts as a "channel" that lets a tiny classical message carry quantum information across.

The wonder is in the bookkeeping. Quantum mechanics forbids many things — copying, exact measurement of unknown states, faster-than-light signaling. Within those rules, it allows this surprising thing. The same rules that prevent quantum cloning also enable quantum teleportation. They are two faces of the same physics.

## Where to go deeper

- Bennett, Brassard, Crépeau, Jozsa, Peres, Wootters, *Teleporting an unknown quantum state via dual classical and Einstein-Podolsky-Rosen channels*, PRL 1993. The defining paper.
- Nielsen and Chuang, *Quantum Computation and Quantum Information*, Section 1.3.7. Standard textbook treatment.
