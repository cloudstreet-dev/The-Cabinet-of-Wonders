# Acoustic cryptanalysis

A computer makes noise — high-pitched whining and ticking from the voltage regulators on the motherboard, audible only to a microphone an inch or two away. The noise depends on what the CPU is computing. Differences in computation produce differences in current draw at the regulator, which produces differences in the regulator's mechanical vibration, which produces differences in the sound. By recording that sound while the CPU performs an RSA decryption, you can recover the private key.

Genkin, Shamir, and Tromer demonstrated this in 2014 against GnuPG running on commodity laptops. The microphone was a regular phone, held a few inches from the laptop's vent. The signal was a faint coil whine. The full 4096-bit RSA key was extracted in about an hour.

## Why a CPU makes noise

CPUs are powered by voltage regulators ("VRMs"): small switching power supplies on the motherboard that convert 12V from the power supply down to the ~1V that the CPU needs, at hundreds of amperes peak. They do this with switching transistors and an LC filter (inductor and capacitor). The inductor — a coil of wire wound around a ferrite core — physically vibrates when current changes, by the same magnetostriction effect that makes transformers hum.

The inductor's vibration is not at the switching frequency (which is hundreds of kilohertz, above hearing). It is at much lower frequencies — the audible band, dominated by 1 kHz to 20 kHz harmonics. The amplitude depends on how much the current is changing, which depends on what the CPU is doing.

Different operations have different power signatures:

- An idle CPU with HLT instructions running has very low and stable current.
- A heavy floating-point loop has high and stable current.
- A code path that branches frequently between idle and busy has *bursty* current with rich audible spectral content.

Crypto operations, especially those involving modular exponentiation, have characteristic patterns. Each multiplication and squaring is a distinct sequence of microarchitectural events with a corresponding power signature.

## Square-and-multiply leaks the key

Naive RSA decryption uses square-and-multiply with the secret exponent \\(d\\):

```
result = 1
for each bit b of d (from MSB to LSB):
    result = result * result mod n          # always: square
    if b == 1:
        result = result * ciphertext mod n  # only on 1-bits: multiply
```

A 1-bit triggers a multiplication step. A 0-bit does not. The two cases have detectably different acoustic signatures (different number of operations, slightly different memory access pattern). By distinguishing 1-bits from 0-bits in the recorded sound, you read off \\(d\\).

The classical countermeasure (and the one in vanilla GnuPG) was to use a sliding-window exponentiation that batches operations into chunks. This reduces leakage but does not eliminate it; the 2014 attack worked through this defense, exploiting more subtle timing differences in the windowed algorithm.

## What the attack actually does

The Genkin-Shamir-Tromer attack:

1. Record acoustic emissions while the target performs a series of decryptions.
2. Filter the audio to a narrow band (around 35-40 kHz, often above the conscious hearing range of adults but well within microphone range) where the regulator artifacts are clearest.
3. Decryption of carefully chosen ciphertexts yields traces with predictable structure. Compare to a model of expected behavior for various candidate key bits; the consistent fit reveals the key.

The cryptanalysis is, in spirit, a chosen-ciphertext attack with the side channel substituting for the response. Asking the target to decrypt millions of ciphertexts, recording the acoustic emanation each time, and applying GCD-based attacks (similar to those used in fault-injection cryptanalysis) yields the secret factors of the modulus.

The attack worked at distances up to several meters with parabolic microphones, several centimeters with a phone microphone resting on the laptop's hinge.

## A cousin: power-side-channel attacks

Acoustic attacks are essentially indirect power-analysis attacks. The original power-analysis literature (Kocher, Jaffe, Jun, *Differential Power Analysis*, 1999) measures CPU current draw directly, by inserting a small resistor in the power supply and reading the voltage drop with an oscilloscope. This is more precise than acoustic — the microphone adds noise, mechanical resonances, and an indirect coupling — but requires physical access to the power line.

Acoustic attacks work without physical contact. A laptop in the next office over, exposed only through air, can still leak its key.

## Defenses

**Constant-time crypto.** Eliminate data-dependent control flow. Decrypt in a way that performs the same number and type of operations regardless of key bits. Standard for modern crypto libraries (BearSSL, Ring, libsodium, modern OpenSSL with proper compile flags). It does not eliminate power-side-channel leakage entirely, because the data path itself can leak through Hamming-weight-dependent power, but it removes the gross signal that the original attack relied on.

**Blinding.** Pre-multiply the ciphertext by a random value, decrypt, post-divide. Now the attacker cannot choose the ciphertext that is actually exponentiated; the relationship between key bits and emissions is randomized.

**Acoustic shielding.** Computers in sensitive applications can be in soundproofed rooms. The TEMPEST standard for shielded computing facilities anticipated this kind of attack decades ago, though it focused on EM emanations rather than sound.

**Physical countermeasures.** Modify the power-supply design to reduce the current-modulating effect of CPU work. Increase decoupling capacitance; use higher switching frequencies; choose inductors that vibrate less audibly. These shift the signal but rarely eliminate it.

In practice the right answer is constant-time, blinded crypto, in software, written by people who treat side channels as part of the threat model.

## Why this is wonder, even though it is unsettling

The image: a phone sitting on a table near a laptop. The laptop is doing what laptops do. To anyone watching, nothing is happening. But the phone, recording, captures a faint coil whine, ten seconds long, and from that whine — given enough time, a known input pattern, and the computational cost of cryptanalysis — the private key falls out.

The wonder is that the side channel exists at all. CPU designers, operating-system designers, cryptography library authors all spent decades constructing a digital abstraction sealed off from the analog world. The abstraction works for everything *inside* the machine: instructions execute, memory is private, kernel is isolated. But the analog world is the substrate underneath, and the substrate is leaky. Power flows. Inductors vibrate. Microphones hear. The digital world's secrets, given enough physical correlation, are observable.

It is the strongest existence proof we have that "isolation" inside a machine is not really isolation. The machine is connected to the room.

## Where to go deeper

- Genkin, Shamir, Tromer, *RSA Key Extraction via Low-Bandwidth Acoustic Cryptanalysis*, CRYPTO 2014. The defining paper, with audio samples online.
- Kocher, Jaffe, Jun, *Differential Power Analysis*, CRYPTO 1999. The classical (direct) power-analysis paper.
