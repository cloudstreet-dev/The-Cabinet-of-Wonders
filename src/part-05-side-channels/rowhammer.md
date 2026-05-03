# Rowhammer

If you read the same row of DRAM rapidly enough, the rows next to it will start flipping bits. Not the row you are reading. The neighbors. The hardware does not detect this; the operating system does not see it; the read access itself is a perfectly legitimate user-mode operation that touches no privileged memory at all. But the act of reading creates a small electrical disturbance in the silicon, and that disturbance, repeated millions of times per second, accumulates until charges leak across the cell barriers and a 0 in a neighboring row becomes a 1, or a 1 becomes a 0.

Once the attacker can flip arbitrary bits in memory they do not own, they can do nearly anything: take over the kernel, break out of a sandbox, escalate privileges, escape a virtual machine. The first practical Rowhammer-based exploit (Seaborn and Dullien, 2015) escaped Google's NaCl sandbox by flipping bits in page tables.

## Why DRAM works

A DRAM cell is a capacitor and an access transistor. The capacitor holds a tiny charge representing a bit (charged = 1, uncharged = 0, conventionally). To read, the capacitor's charge is sensed by a *bitline* shared with all other cells in its column. To write, the bitline is driven to the desired voltage and the access transistor latches the cell.

The capacitor is leaky — it loses charge over time, due to thermal effects and parasitic resistance. Modern DRAM is *refreshed* every 64 ms (or 32 ms in some standards) by reading every row and rewriting it, restoring the charge. The refresh is invisible to software but mandatory.

The cells are organized in *rows*. Reading or writing requires *activating* a row: pulling its wordline high, which connects all cells in that row to their bitlines simultaneously. After activation, the bitlines are sensed (read) or driven (write), and then the row is *closed* — wordline back low, charges restored.

## What rowhammer does

Activating a row toggles the wordline voltage. The wordline is a metal trace running across the chip; nearby wordlines (rows above and below physically) are coupled to it through parasitic capacitance. When the activated wordline pulls high, the neighbors get a small voltage kick. When it falls back low, the neighbors get another kick.

Each kick is tiny. Refresh restores any charge lost. But if you repeatedly activate the same row, fast enough, the kicks happen faster than refresh can recover. Each kick leaks a fraction of charge from cells in the neighboring rows. After enough kicks, a cell crosses its detection threshold and is read incorrectly. That is a bit flip.

## The exploit pattern

```c
// pseudocode for the original Rowhammer
volatile char* row_a = ...;  // attacker-readable address
volatile char* row_b = ...;  // a different row, same bank, separated from row_a
                             // by exactly one or two rows containing victim data

for (int i = 0; i < 1000000; i++) {
    *row_a;        // read - activates row_a
    *row_b;        // read - activates row_b, forces row_a out
    clflush(row_a);  // x86 instruction: flush from CPU cache,
                     // forcing actual DRAM access on next read
    clflush(row_b);
}
```

The pattern is *double-sided* hammering: rows on both sides of the victim row are activated repeatedly. The victim's wordline gets two kicks per cycle. After tens of thousands of cycles per row pair (a fraction of a second on commodity hardware), bits flip.

`clflush` is critical. Without it, the second access to `row_a` would hit the CPU cache and never reach DRAM. The instruction is unprivileged (intentionally — it's a performance hint for software) and forces eviction. With it, every loop iteration triggers a real DRAM activation.

There are variants: *one-location hammering* (just keep accessing row_a, with cache flushes); *many-sided hammering* (multiple aggressor rows targeting one victim); *Tridge* / *TRRespass* (defeating in-DRAM mitigations); *Half-Double* (bypassing nearest-neighbor defenses by hammering rows two apart); *RAMBleed* (using bit-flip patterns to read secrets, not write them).

## Mapping virtual to physical to DRAM

To attack a specific victim — say, a page-table entry at physical address \\(p\\) — the attacker needs to find virtual addresses that map to *DRAM rows physically adjacent to* the row containing \\(p\\). This requires reverse-engineering the DRAM addressing of the system: which physical-address bits map to bank, row, column, in what order, with what XOR-based hash functions.

This was once thought to be a barrier. It turned out to be straightforward. DRAMA (Pessl et al., 2016) reverse-engineers DRAM mapping with timing measurements: hammer pairs of physical addresses, measure access timing, infer same-bank conflicts, deduce the addressing function. Once known, mapping is deterministic per CPU model.

So the attacker can convert a virtual address to a DRAM row, find virtual addresses one or two rows above and below, and hammer.

## Targeting the kernel

The original Seaborn-Dullien NaCl exploit:

1. The attacker is JIT'd code in NaCl, allowed to flush cache and access big chunks of memory.
2. They map a large region (~1 GB) to find candidate vulnerable bit positions — DRAM cells that flip reliably under hammering.
3. They convince the OS to allocate page-table entries (PTEs) at those exact physical locations. (The OS allocates page tables on demand from the physical-page pool; by carefully timing allocations, the attacker can position PTEs.)
4. They hammer to flip a bit in a PTE that controls *whether the page is writable*. Now they have writable access to a page they should not have, including page tables themselves. Write to page tables. Map any physical address. Read kernel memory. Game over.

The whole exploit is a few hundred lines of NaCl-allowed C code.

## Mitigations and their inadequacy

**TRR (Target Row Refresh)**: DRAM internally tracks "frequently activated" rows and silently issues extra refreshes for their neighbors. Aimed to be invisible to software. Defeated by TRRespass (Frigo et al., 2020) which used many-sided patterns the TRR mitigation did not anticipate. Defeated again, repeatedly, by Half-Double, Blacksmith, and other follow-on patterns.

**ECC (Error Correcting Codes)**: error correction at the DRAM controller. Single-bit flips are corrected, double-bit flips detected. Standard ECC defeats trivial Rowhammer but not multi-bit Rowhammer (ECCploit, 2019, demonstrated three-bit flips in a single word, undetectable by SECDED).

**RowHammer-Aware Refresh**: more frequent refresh of all rows. Costs power and bandwidth. Unattractive in datacenter DRAM where capacity and density are paramount.

**On-die ECC**: newer DRAM standards (DDR5, LPDDR5) include integrated ECC. Helps but does not eliminate.

The fundamental problem: Rowhammer is a property of the *physics* of cramming more cells into less silicon. As DRAM density rises (45 nm to 32 nm to 10 nm cells, and shrinking), cells become smaller and parasitic coupling stronger. Rowhammer thresholds — number of activations per refresh window to flip a bit — drop monotonically with density. DDR3 needed 1M+ activations per row; DDR4 sometimes flips at 50K; DDR5 may be even lower.

So new DRAM is more vulnerable, not less. The mitigations are arms races.

## What it leaks beyond bit flips

**RAMBleed** (Kwong et al., 2020): use Rowhammer-style hammering not to flip bits but to *read* them. Activated rows leak data via the bitlines into the sense amplifiers; nearby rows share the bitline structure. Carefully crafted hammering allows inferring secret data in adjacent rows from the access pattern, without requiring a bit flip. SSL keys and other secrets in memory have been read this way.

## The wonder, ungentle

Rowhammer is the most physical security vulnerability in modern computing. It does not exploit a software bug, a protocol weakness, or a configuration mistake. It exploits the fact that the cells in DRAM are made of atoms, and the atoms interact electrostatically with each other, and the silicon manufacturing processes have shrunk past the point where physical isolation alone is sufficient. The defenders are fighting against the laws of physics, with each fix delayed by the next density shrink.

The exploit is short. The mitigations are hard. And the trend is going the wrong way: more density, more vulnerability. Every cloud provider, every operating system kernel, every memory-controller team has been forced to accept that DRAM is a cooperative layer rather than a black-box storage primitive, and that "I read my own memory rapidly" is a vector that touches everything.

## Where to go deeper

- Kim et al., *Flipping Bits in Memory Without Accessing Them: An Experimental Study of DRAM Disturbance Errors*, ISCA 2014. The paper that named the phenomenon.
- Seaborn and Dullien, *Exploiting the DRAM Rowhammer Bug to Gain Kernel Privileges*, Project Zero 2015. The first practical exploit.
- Frigo et al., *TRRespass: Exploiting the Many Sides of Target Row Refresh*, S&P 2020. Defeating modern in-DRAM mitigations.
