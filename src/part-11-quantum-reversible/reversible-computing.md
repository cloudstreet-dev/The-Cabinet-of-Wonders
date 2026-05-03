# Reversible computing

You can build a complete computer — universal Turing machine, all standard algorithms — out of gates that are *logically reversible*: every output uniquely determines the input. No information is lost during computation. As a consequence, the laws of thermodynamics permit such a computer to dissipate, in principle, *zero* energy as heat. Modern CPUs dissipate energy primarily because they erase information; a reversible CPU does not erase, so it does not, in principle, need to dissipate.

The connection between information and thermodynamics was sharpened by Landauer (1961): erasing one bit of information costs at least \\(k_B T \ln 2\\) of energy. Reversible computing avoids erasure. Bennett (1973) showed that any computation can be done reversibly, which removed the "but real computers must erase" objection. The pieces are theoretical for now — practical CPUs lose orders of magnitude more energy to other causes — but the result is one of the deepest connections between thermodynamics and computer science.

## Landauer's principle

Erasing one bit of information requires dissipating at least \\(k_B T \ln 2 \approx 3 \times 10^{-21}\\) joules at room temperature. The argument: the bit before erasure was a state of two equally likely possibilities (entropy \\(k_B \ln 2\\)); after erasure, it is fixed (entropy \\(0\\)). The decrease in entropy of the bit must be matched by an increase in the entropy of the environment (second law of thermodynamics), which means heat \\(\geq T \cdot \Delta S = k_B T \ln 2\\).

This is the *minimum*. Real CPU gates dissipate roughly \\(10^4\\) times this, due to switching losses, leakage, and thermal margins. But Landauer's bound is a hard floor.

For computers that do not erase, the bound does not apply. A reversible computer that retains all its history can in principle dissipate as little energy as you like.

## Logically reversible gates

A standard logic gate (AND, OR, NAND, etc.) is generally not reversible: knowing the output does not let you reconstruct the inputs. AND has two inputs (4 possible) but only one output (2 possible) — information is destroyed.

A *reversible* gate has the same number of inputs and outputs and is bijective. The output uniquely determines the input.

Examples:
- **NOT**: input \\(x\\) → output \\(\bar{x}\\). One bit in, one bit out. Reversible.
- **CNOT (controlled NOT)**: two inputs \\((a, b)\\) → two outputs \\((a, a \oplus b)\\). The first wire passes through unchanged; the second is XOR'd with the first. Reversible.
- **Toffoli (controlled-controlled-NOT)**: three inputs \\((a, b, c)\\) → three outputs \\((a, b, c \oplus (a \wedge b))\\). The third wire is XOR'd with \\(a \wedge b\\). Reversible. Toffoli is *universal* for classical reversible computing — any reversible function on bits can be built from Toffoli gates plus ancillae.

So reversible computation is doable with a single universal gate type.

## Embedding any computation in reversible form

Bennett (1973): given any computation \\(f(x) = y\\), build a reversible version \\((x, 0) \mapsto (x, y)\\). The output keeps a copy of the input alongside the result. This is reversible (the input is recoverable from the output) and computes \\(f\\).

The cost is *garbage*: the computation may produce intermediate values that are kept in the output. Bennett's compression: do the computation forward, copy the result to an output register, then run the computation *backward* to clean up the intermediate state. This recovers the input and leaves only the result, with no garbage. The full cost is roughly twice the time of the original computation, plus space for the input and the result.

So any irreversible computation can be made reversible with at most polynomial overhead. The theoretical existence of low-energy computing is not blocked at the algorithmic level.

## Why this matters in quantum computing

Quantum computation is intrinsically *reversible*: every quantum gate is a unitary, and unitaries are bijective. Quantum circuits are sequences of unitaries; they are exactly reversible computations.

So reversible classical computation is the *classical sub-theory of quantum computation*. The Toffoli gate is universal for classical reversible computing; it is also a gate in many quantum-computing universal gate sets. To run a classical algorithm on a quantum computer, you first transform it to reversible form (using Bennett's construction), then implement the reversible gates as quantum gates.

This is not optional — quantum computers cannot perform irreversible operations as part of their unitary evolution. The conversion to reversible form is mandatory. Bennett's 1973 result was, in retrospect, providing the algorithmic infrastructure for quantum computing decades before quantum computing existed.

## Maxwell's demon and information

The Landauer-Bennett picture finally resolved a 150-year-old puzzle: Maxwell's demon. The demon, in Maxwell's 1867 thought experiment, is a microscopic being that selectively opens a door between two gas chambers, sorting fast molecules to one side and slow ones to the other. This decreases the gas's entropy, in apparent violation of the second law.

Resolution (Landauer-Bennett-Bennett): the demon must measure each molecule's speed. Measurements store information in the demon's memory. Eventually the memory fills up. To continue working, the demon must *erase* its memory. Erasure costs energy (Landauer). The energy cost of erasure precisely cancels the work the demon would have extracted from sorting.

So the second law is preserved, *because of* the cost of information erasure. Information theory is a piece of thermodynamics.

This is a wonder in itself: the cost of an *abstract* operation (forgetting a bit) is a *physical* quantity (joules). Information is physical.

## Real-world implementations

Strict reversible CPUs (no irreversible operations, all-bijective hardware) have been built only as research demonstrations. The largest are Pendulum (MIT, 1990s) and various academic prototypes. They demonstrated functional reversible processors but did not approach the theoretical low-energy limit, because real circuits have many sources of dissipation beyond information erasure.

Adiabatic CMOS — circuits that switch slowly, using charge-recycling power supplies that minimize \\(C V^2 / 2\\) loss — have been used in some ultra-low-power applications. They are partially reversible and do save energy in regimes where switching loss dominates.

Modern interest in reversible computing has come from:
- Quantum computing (where all logic is reversible by mandate).
- Cryogenic computing for superconducting qubit control electronics, where heat dissipation matters at millikelvin temperatures.
- Neuromorphic and asymptotically-zero-energy computing schemes for ultra-low-power devices.

For mainstream CPUs running gigahertz workloads at room temperature, reversibility is not the bottleneck — switching losses, leakage, and clock distribution dominate. The Landauer bound is many orders of magnitude below current dissipation. Reversible computing is a longer-term direction, important if exponential improvements continue past where current physics caps the conventional architecture.

## The wonder

The act of erasing a bit of information is, by the second law of thermodynamics, *physically* costly. Reversible computing demonstrates that this cost is avoidable: any computation can be done without erasure, and so in principle without any thermodynamic minimum dissipation.

The connection between information and thermodynamics is not metaphorical. It is quantitative. The bit is real, the energy is real, the conversion factor (\\(k_B T \ln 2\\)) is calculable. Compute without erasing, you can in principle compute without dissipating energy. The bound is physics, not engineering.

The same physics implies the equivalence: a reversible classical computer is mathematically the classical sub-theory of a quantum computer. Quantum computing was, in some sense, "reversible computing extended to allow superposition." Whether the practical engineering ever drives down to the Landauer limit remains to be seen; the theoretical foundation is settled.

## Where to go deeper

- Bennett, *Logical Reversibility of Computation*, IBM Journal of Research and Development 1973. The defining paper.
- Frank, *Reversible Computing FAQ*, 2017. Modern overview of the engineering and theoretical state.
