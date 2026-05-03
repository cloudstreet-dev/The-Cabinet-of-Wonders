# Power analysis

A smart card running an AES decryption draws current that depends, very slightly, on the value of the secret key bits being processed. The differences are tiny — microamps on top of milliamps of average current, on a microsecond timescale. With a small resistor in the power line and an oscilloscope, you can record the current trace. With statistical analysis across a few hundred decryptions, you can recover the key.

The original technique (Kocher, Jaffe, Jun 1999) extracted DES and AES keys from smart cards in widespread commercial use. It was, more than any other side-channel result, the demonstration that "constant-time" was not enough — the algorithms had to be *constant-data-pattern*, and even then, hardware leaks remained.

## Why power varies with data

A CMOS gate dissipates negligible static power; almost all energy goes into switching transitions. The dynamic power per gate is

\\[ P = \alpha \cdot C \cdot V^2 \cdot f \\]

where \\(\alpha\\) is the fraction of clock cycles in which the gate transitions, \\(C\\) is its capacitance, \\(V\\) is voltage, \\(f\\) is frequency. The variable across operations is \\(\alpha\\): how many gates flip from 0 to 1 (or 1 to 0) during this clock cycle.

For a 32-bit register being written, the number of transitions is the *Hamming distance* between the previous value and the new value. A write of 0x00000000 over 0x00000000 has 0 transitions; a write of 0xFFFFFFFF over 0x00000000 has 32. The current spike scales linearly with the bit count.

So the current trace is a linear function (plus noise) of bit transitions in the data path. If part of the data path is the secret key, the current trace contains a linear function of the key.

## Simple Power Analysis (SPA)

The first kind of attack: just read the trace. For poorly-implemented crypto, the trace shows the algorithm's structure visibly:

- A square-and-multiply RSA implementation has distinct *square* and *multiply* operations. The multiply happens only on 1-bits of the secret exponent. The two operations have different durations and different power signatures. Reading off the trace, you read off the exponent.

```
Power
trace                       ____         ____
       __  _  __     __    /    \  __   /    \
   ___/  \/ \/  \___/  \__/      \/  \_/      \___ 
   sq  sq sq sq mul sq sq mul sq sq sq mul sq sq sq mul
   0   0  0  0   1  0  0   1  0  0  0   1  0  0  0   1
                          
   exponent bits visible in trace: 0000 1 00 1 000 1 000 1
```

Defense: use a constant-time exponentiation algorithm where every iteration performs both a square and a multiply, regardless of the key bit; throw away the multiply result if the bit is 0. Eliminates the SPA leak.

This brings us to differential analysis.

## Differential Power Analysis (DPA)

Constant-time code does not have visible algorithm-flow in its trace. But it still has data-dependent power. A 32-bit AES round produces 32 bits of state; the wire transitions of that round depend on the bits' values. The current spike in the round is correlated with the value of the state.

DPA exploits this with statistics.

The key observation: if you know what plaintext was encrypted, and you know what AES does, you can predict (for each candidate value of a key byte) what an intermediate value would be. The actual current trace will be slightly more correlated with the *correct* prediction than with wrong predictions.

DPA procedure (Kocher et al.):

1. Collect traces from \\(N\\) encryptions with known plaintexts. \\(N\\) might be a few hundred to a few thousand.
2. Pick one byte of the key to attack. There are 256 candidates.
3. For each candidate \\(k\\):
   - Compute the predicted intermediate value (e.g., the output of the first S-box) for each plaintext: \\(v_i = S(p_i \oplus k)\\).
   - Bucket the traces by the value of one bit of \\(v_i\\) (say, the LSB).
   - Compute the difference between the average trace of "bit = 0" and "bit = 1" buckets.
4. The correct \\(k\\) yields a difference trace with a sharp spike where the predicted bit is being computed (and zero elsewhere). Wrong \\(k\\) values yield random-looking differences (no specific moment when wrong predictions correlate).

Plotting all 256 difference traces, the correct one has a visible spike. Recover the key byte. Repeat for each byte. Total cost: a few hours of compute on a few thousand traces.

## Correlation Power Analysis (CPA)

A refinement (Brier, Clavier, Olivier 2004): instead of bucketing by predicted bit, *correlate* the traces with a power model. The power model is "Hamming weight of the predicted intermediate" — assume current scales with HW of the wire being driven.

Compute Pearson correlation between the trace samples and the predicted HW for each candidate \\(k\\). The correct \\(k\\) gives the highest correlation. Mathematically cleaner than DPA, requires fewer traces (often hundreds suffice).

## Defenses

**Algorithmic-level**:

- **Constant-time code**: removes SPA. Necessary first step.
- **Masking**: split each intermediate value into shares. \\(v = v_1 \oplus v_2\\) where \\(v_1\\) is uniform random and \\(v_2 = v \oplus v_1\\). Operate on shares; combine only at end. The current trace correlates with shares, which are randomized per execution, so the average correlation with the true intermediate is zero. Higher-order DPA (look at correlations of products of trace samples) defeats first-order masking; second-order masking defeats first-order DPA, etc. — an arms race.
- **Hiding**: add random delays, dummy operations, randomized order of independent steps. Weakens but does not eliminate.

**Hardware-level**:

- **Dual-rail logic**: each bit is encoded on two wires that always change opposite ways. Total transitions per cycle are constant regardless of data.
- **Asynchronous logic**: no global clock; current draw is more uniform.
- **On-chip current regulators with feedback**: actively flatten current spikes.
- **Faraday-shielded chip packaging with on-chip power averaging capacitors**: smooths out the current externally, leaving only the time-averaged value.

For deployed chips, the answer is usually: certify against side channels under a specific Common Criteria EAL or FIPS 140 level, which mandates resistance to specified attacker models with measured trace counts.

## What this implies for crypto on small devices

Smart cards, secure elements, IoT chips, hardware wallets, EMV terminals — all of these run cryptographic operations on limited hardware where physical access is plausible (the attacker has the card). All of them have to be designed against power analysis. The implementations cost a factor of 2-10× more area and 2-5× more power than naive ones. The certification cost is enormous (months of testing per chip).

Crypto on big computers (servers, laptops) is less affected because the attacker rarely has direct physical access to the power line, and the noise of millions of unrelated gates makes individual operations harder to isolate. But not immune — see *Acoustic cryptanalysis*, which is essentially a remote acoustic version of power analysis.

## EM-side-channel attacks

A close relative: *electromagnetic emanations* from the chip carry the same data-dependent information that the power line carries. A small loop antenna or magnetic probe held against the chip picks up signals correlated with internal state. The math is the same as DPA. The advantage to the attacker: no need to insert a resistor or modify the device; the radiation can be received with a probe touching the package or even at small distance. The technique is sometimes called *EM analysis* and is widely used in evaluation labs.

## The wonder, with a hard edge

A cryptographic algorithm is mathematically secure: it has been analyzed by the best cryptographers; the best known attacks require exponential time. An implementation of that algorithm is, in nearly every plausible setting, not secure. Given even modest physical access — a resistor in the power line, a probe near the chip, a microphone on the desk — the algorithm's secrets fall out in seconds to hours.

The mathematical security and the implementation security are different categories. A perfect mathematical attack model and a correct implementation are not the same; the attack model assumes the adversary has only the input/output relation, but the *implementation* exposes the entire instantaneous physical state to anyone willing to measure it. Power analysis is the cleanest, oldest, most thoroughly understood example of this gap.

After 25 years of research, the field is mature: protections exist, evaluation methods exist, and well-engineered hardware can provide effective resistance up to specified attack budgets. But the underlying principle — that data-dependent power consumption is universal in CMOS — remains. The defense is always layered, partial, and probabilistic.

## Where to go deeper

- Kocher, Jaffe, Jun, *Differential Power Analysis*, CRYPTO 1999. The defining paper.
- Mangard, Oswald, Popp, *Power Analysis Attacks*. The textbook (2007).
