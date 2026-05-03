# Spectre and Meltdown

A modern CPU does not run your code in the order you wrote it. It speculatively executes ahead of conditional branches, before it knows whether the branches will be taken. If the speculation turns out to be wrong, the CPU rolls back the architectural state — registers and memory — and pretends the speculation never happened. The recovery is so clean that, from software's perspective, nothing happened.

Except for the cache. The CPU also has caches, and the recovery does not roll those back. The cache state still reflects what the speculation accessed. If your program can measure cache state, it can detect what the speculation did. And if you can goad the speculation into doing things you are not allowed to do — read past array bounds, dereference unmapped pages — the cache becomes a side channel for data you should not be able to see.

That is the entire trick. It works against essentially every modern out-of-order CPU. It got its own keynote at every operating systems conference for two years. Every cloud provider on Earth had to redesign their infrastructure.

## Out-of-order execution and speculation

Modern CPUs (Intel, AMD, ARM, IBM) execute instructions out of order. Behind every memory access lies a long latency — main memory might be 200 cycles away. Rather than wait, the CPU keeps a window of upcoming instructions, schedules them as their inputs become available, and commits results in program order to maintain the architectural illusion of sequential execution.

To keep the pipeline full, the CPU also predicts the outcome of conditional branches before they execute. The branch predictor is a small machine-learning model trained on the program's recent branching history. When it predicts "likely taken," the CPU speculatively executes the taken side; when it predicts "likely not taken," the not-taken side. The speculative results sit in private buffers until the branch resolves. If the prediction is right, results are committed. If wrong, the speculative work is squashed.

This is incredibly effective: 90%+ accurate branch prediction means most code runs at near-perfect pipeline utilization. Without it, modern CPUs would be far slower.

## Spectre v1: bounds-check bypass

Consider this code:

```c
if (x < array1_size) {
    y = array2[array1[x] * 256];
}
```

Sound code: bounds-checks `x` before accessing `array1`. If the predictor sees the check usually pass, it learns to predict "branch taken" — not because of any malice, just because that is the common case.

Now an attacker calls this function with `x` *out of bounds*. The branch predictor, trained on prior in-bounds calls, predicts "taken" anyway. Speculative execution reads `array1[x]` (out of bounds, possibly past the end of the array into protected memory) and uses that value to index `array2`. The line of `array2` corresponding to the secret value gets pulled into the cache.

The branch resolves: the predictor was wrong, the architectural state is rolled back, no value of `array1[x]` is exposed in any register. But `array2`'s cache line is still warm. The attacker measures access times to all 256 possible cache lines (the speculation indexed by `array1[x] * 256`); the warmest one corresponds to the byte they just read out of bounds.

```
attacker code (in same process as victim function above):
  flush all 256 lines of array2 from the cache
  call victim(out_of_bounds_x)
  for each i in 0..255:
      time the access to array2[i * 256]
  the fastest one is array1[out_of_bounds_x]
```

Repeat for each byte of the target buffer. The attacker reads the entire address space of the process, byte by byte, by repeatedly calling a benign-looking function with carefully crafted inputs.

## Spectre v2: branch target injection

Spectre v1 exploits direct branch prediction. Spectre v2 exploits *indirect* branches — function pointers, virtual method calls, returns. The branch predictor for indirect branches has limited entries; an attacker can poison them to redirect speculation into a *gadget* of the attacker's choosing.

Process A's "indirect branch from address X" can have its predictor entry poisoned by Process B's "indirect branch from address X" (in some implementations, the BTB indexes by virtual address modulo a hash, ignoring address-space identifiers). So a guest VM can poison the host hypervisor's branch predictor, causing the host to speculatively execute attacker-chosen code on attacker-chosen data, leaking it via cache.

Cloud isolation models — different tenants on the same physical CPU — were directly affected.

## Meltdown: dereferencing kernel memory from user space

Meltdown is conceptually distinct from Spectre, though they shipped together. On affected CPUs (mostly Intel), out-of-order execution allows a user-mode load instruction to *speculatively* read from a kernel address before the privilege check fails. The architectural exception is delivered later — after the load has issued. By then, the speculative load has used the kernel byte's value as a cache-line index, and the side channel is open.

```
mov rax, [kernel_address]   ; speculatively executed; eventually faults
mov rbx, [user_array + rax * 256]   ; speculatively reads user_array[byte * 256]
                                    ; pulls a cache line in
[fault delivered, registers rolled back]
[but the cache line is still warm]

attacker measures access time on user_array[0..255 * 256]; fast one is the kernel byte
```

The result: a user-mode process can read arbitrary kernel memory, a couple of bytes per millisecond, on affected CPUs. On a multi-tenant cloud, the kernel often contains the entire memory of the host (via the kernel's direct-physical-memory mapping), so this leaks all VM memory.

The fix at the OS level is *Kernel Page-Table Isolation* (KPTI on Linux, KVA Shadow on Windows): map only a tiny stub of the kernel into the user-mode page tables, so attempting to load from kernel addresses faults at translation time, before any speculation can complete. KPTI introduced 5–30% performance overhead on syscall-heavy workloads, depending on workload. AMD CPUs were not affected by Meltdown because their privilege check happens earlier in the pipeline, before the speculative load issues.

## What the mitigations cost

The defenses fall into several categories:

**Hardware**: subsequent CPU generations (Intel Coffee Lake refresh and later, AMD Zen 2 and later) added in-silicon mitigations: enhanced isolation of branch predictor state across modes, defenses against speculative dereference of unmapped or kernel pages, etc. Modest performance cost.

**Microcode**: Intel and AMD shipped microcode updates that add fences (`LFENCE`, `IBRS`, `IBPB`) to flush or restrict the predictor state across context switches. Heavy performance cost on syscall-heavy workloads (10–30%).

**Compiler**: `retpoline` is a compile-time mitigation for Spectre v2 indirect branches. Replaces every indirect call with a sequence that turns it into a return-stack-managed loop, defeating the branch target injection. Cost: indirect calls become slow (1–5 ns extra each); negligible on most workloads, painful on hypervisor and kernel hot paths.

**OS / Hypervisor**: KPTI for Meltdown; SMAP/SMEP enforcement; explicit speculation barriers in critical sections (e.g., array access in syscall handlers).

The combined cost varies by workload. A web server with many short syscalls might be 20% slower; a tight numeric kernel, 0–5% slower. Cloud providers absorbed billions of dollars of effective compute loss.

## Why this is permanent

The fundamental problem is architectural, not a bug. *Speculative execution that observably affects microarchitectural state* is the entire technique behind modern CPU performance. Removing speculative execution would set CPU performance back two decades.

So the workaround is to make the speculation as harmless as possible: prevent it from crossing privilege boundaries (KPTI, BTB isolation), narrow the kinds of state it can affect (cache flushing, retpolines), and audit code for gadgets that the speculation could exploit.

But the underlying primitive — speculation that touches the cache, with the cache being measurable — is still there. New variants (Foreshadow, ZombieLoad, RIDL, Fallout, MDS, LVI, Retbleed, ZenBleed, Inception, Downfall) keep appearing as researchers find new ways to coax microarchitectural state to leak. The patches keep coming.

## The wonder, with a sharp edge

The performance gains from speculative execution are real and immense. Modern CPUs are vastly faster than 1990s in-order CPUs partly because they predict and execute past every conditional branch. The CPU designers built a pure speedup, with no exposed semantic effect — the architectural state was always rolled back; speculation was strictly invisible to programs.

Except it was not invisible. The cache is microarchitectural, but it is observable through timing. Any feature that affects observable timing is a leak channel. The CPU designers had spent 30 years ensuring speculative execution had no architectural effect; what they did not (and could not) ensure was that it had no microarchitectural effect, because the cache *is* the microarchitectural effect they were trying to exploit for performance.

Spectre and Meltdown are wonders because they are an existence proof that *the entire performance regime modern CPUs operate in* contains an unfixable information leak. The CPU literally cannot do what it does — speculate, execute, roll back — without leaving traces in the cache that a sufficiently clever attacker can read. The only way to be safe is to slow down: insert fences, isolate state across contexts, accept the performance cost. Two decades of CPU optimization had quietly built a cathedral of speculation, and one cleverly-constructed gadget can read everything inside it.

## Where to go deeper

- Kocher et al., *Spectre Attacks: Exploiting Speculative Execution*, S&P 2019. The defining paper.
- Lipp et al., *Meltdown: Reading Kernel Memory from User Space*, USENIX Security 2018. The Meltdown paper.
- Mark Brand and Jann Horn's *Project Zero* writeups, 2018. The contemporaneous engineering walkthrough.
