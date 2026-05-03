# Superdense coding

The mirror image of quantum teleportation. There, you transmitted one *qubit* of information using one entangled pair plus two classical bits. Here, you transmit two *classical bits* of information by sending one qubit (assuming a pre-shared entangled pair). The same resource — entanglement — doubles the capacity of a quantum channel.

Bennett and Wiesner described it in 1992. It is a proof that entanglement, deployed correctly, is a *communication resource*: a pre-shared entangled pair lets you do things with a single quantum channel that you cannot do classically.

## The setup

Alice and Bob share an entangled pair \\(|\Phi^+\rangle = (|00\rangle + |11\rangle)/\sqrt{2}\\); Alice has the first qubit, Bob has the second.

Alice wants to send Bob two classical bits \\(b_1 b_2 \in \{00, 01, 10, 11\}\\). She has only one qubit to send.

## The protocol

Alice applies one of four Pauli operations to *her* qubit, depending on \\(b_1 b_2\\):
- \\(00 \to I\\) (identity, no change).
- \\(01 \to X\\) (bit flip).
- \\(10 \to Z\\) (phase flip).
- \\(11 \to X Z\\) (both).

After Alice's operation, the joint state is one of four orthogonal Bell states:

| \\(b_1 b_2\\) | Operation | Resulting Bell state |
|---|---|---|
| 00 | \\(I\\) | \\(|\Phi^+\rangle = (|00\rangle + |11\rangle)/\sqrt{2}\\) |
| 01 | \\(X\\) | \\(|\Psi^+\rangle = (|01\rangle + |10\rangle)/\sqrt{2}\\) |
| 10 | \\(Z\\) | \\(|\Phi^-\rangle = (|00\rangle - |11\rangle)/\sqrt{2}\\) |
| 11 | \\(X Z\\) | \\(|\Psi^-\rangle = (|01\rangle - |10\rangle)/\sqrt{2}\\) |

Alice sends her qubit to Bob.

Bob now has both qubits and performs a *Bell-basis measurement*: a unitary that maps the four Bell states to the four computational basis states \\(|00\rangle, |01\rangle, |10\rangle, |11\rangle\\), followed by a measurement in the computational basis. He reads two classical bits, which match \\(b_1 b_2\\) exactly.

Two bits transmitted. One qubit physically sent. Plus one consumed entangled pair. Net rate: 2 classical bits per qubit — *twice* the channel capacity of a non-entanglement-assisted quantum channel.

## Why this is not faster-than-light

The qubit physically traveled from Alice to Bob. The transmission is bounded by the speed of light. What is unusual is the *capacity*: the qubit carries 2 bits of information, not 1.

Without the pre-shared entanglement, a single qubit can carry only 1 classical bit (Holevo's bound). With it, 2. The doubling comes from the pre-positioned entanglement, which encoded "information about correlation" between Alice's and Bob's qubit before any communication began.

## Why this is the dual of teleportation

Teleportation: 1 entangled pair + 2 classical bits → 1 qubit transmitted.
Superdense coding: 1 entangled pair + 1 qubit transmitted → 2 classical bits.

The two protocols are inverses, in the sense of resource-counting:

- Teleportation consumes entanglement to convert classical bits into a quantum state.
- Superdense coding consumes entanglement to convert a quantum state (Alice's operation) into classical bits.

Both protocols treat entanglement as a fungible resource that can be expended to perform conversions between classical and quantum communication.

## Optimality

Holevo's theorem: a single qubit (without prior entanglement) carries at most 1 classical bit.

Superdense coding doubles this *with the help of pre-shared entanglement*. It is also tight: you cannot do better than 2 classical bits per qubit, even with arbitrary entanglement.

So pre-shared entanglement gives a factor-of-2 speedup over the classical channel capacity. This factor of 2 is the precise "amount" of entanglement gained from one Bell pair, in the channel-capacity sense.

## What it implies for quantum networks

For a quantum network with established long-distance entanglement (via repeaters, satellites, etc.), classical communication can be made twice as efficient. In the limit where all the entanglement you need is already on hand, you double the bandwidth of any classical channel.

Real quantum-key-distribution (QKD) networks use this kind of resource accounting. Capacity, error rate, and security all depend on how much entanglement is available in the system.

For practical quantum networks, the engineering bottleneck is the *creation and distribution* of entangled pairs, not the use of them. Once you have entanglement, doubling channel capacity is straightforward.

## Why it has been less famous than teleportation

Teleportation captures the imagination — "transmit a quantum state across the galaxy" sounds more dramatic than "double a classical channel's bandwidth." But the two protocols are equally fundamental and equally surprising; they are dual to each other in the resource theory of quantum communication.

In an applications sense, superdense coding is more directly useful for *increasing bandwidth*. Teleportation is more directly useful for *converting between physical media* (e.g., transferring a qubit's state from a memory to a flying photon). Different jobs, same underlying physics.

## The wonder

Two protocols that look like opposite operations — converting classical to quantum, and quantum to classical — both consume a single pre-shared entangled pair and both achieve their effect via local operations and a single round of communication. The symmetry between them is the operational meaning of *entanglement as a resource*: it can be spent to enable conversions in either direction.

You could be forgiven, on first encountering quantum mechanics, for thinking entanglement is a quirky correlation phenomenon, useful only for understanding subatomic experiments. Superdense coding shows it is much more: it is a *budget* you can spend, in a quantifiable way, to perform communication tasks classical systems cannot perform. One Bell pair = factor-of-2 channel doubling, or one qubit's worth of teleportation, or other equivalent conversions.

The wonder is the conservation. The amount of "useful work" you can extract from a Bell pair is a precise, fixed quantity. The protocols realize different ways of spending it. There is no protocol that gets more out of one Bell pair than the dual constructions; the bound is tight, the resource theory is exact.

## Where to go deeper

- Bennett and Wiesner, *Communication via one- and two-particle operators on Einstein-Podolsky-Rosen states*, PRL 1992. The defining paper.
- Nielsen and Chuang, *Quantum Computation and Quantum Information*, Sections 2.3 and 12. Standard textbook.
