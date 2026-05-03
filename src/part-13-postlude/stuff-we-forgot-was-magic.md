# The Stuff We Forgot Was Magic

The opening of this book argued that you live inside black magic and have forgotten it is magic. Part I was a few specific examples — public-key cryptography, GPS, TCP, lossy compression. By now, having walked through twelve parts of further wonders, you have a richer picture. So this last entry comes back around. Not to summarize. To name a few more pieces of ambient magic that the cabinet's walk has earned the right to discuss, and to ask why the wonder fades.

## Some more ambient magic

A few examples that did not get full entries because their construction is not strange — only their existence in your daily life is.

**Voltage regulation in your laptop's power supply.** A switching converter takes 12V from the wall and delivers 1V to the CPU, at 100A peak, at 95% efficiency, controlling the output to within 1% of target across load swings of three orders of magnitude. The control loop runs at megahertz, in hardware. If it failed by 10%, the CPU would burn out. It does not fail.

**Real-time scheduling in the kernel.** A modern operating system juggles thousands of processes, dispatches them to a few cores, preempts on every interrupt, balances priority and fairness, and never loses your keystroke even when the CPU is 99% utilized. The same kernel runs on a fridge, a phone, a 10,000-core server.

**Caching at every level.** Between you and the CPU, between the CPU and DRAM, between DRAM and disk, between disk and network, between any client and any server. Each cache uses LRU, LFU, or some modern variant; each has hit rates above 90% on typical workloads despite no domain knowledge of what you are doing. The set of all cache layers in a modern web request is dozens deep, and the latency stack is engineered against the speed of light.

**Garbage collection.** Programs allocate memory continuously; some collector identifies what is still reachable; everything else is freed. Modern GC pauses are sub-millisecond on multi-gigabyte heaps. The collector runs concurrently with the program, with a write barrier maintaining invariants, with dozens of generations and regions, on systems older than most working programmers.

**The C standard library and its unix descendants.** Forty years of conventions about how programs talk to the operating system. `malloc`, `printf`, `read`, `write`, `socket`. They run on every operating system. They will outlive us all.

**Floating-point arithmetic with NaN and infinity.** A correctly-rounded IEEE 754 implementation across all CPU manufacturers. `0.1 + 0.2 != 0.3` is famous; less famous is that it gives *exactly* the same answer on every conformant machine. A 1985 standard whose semantics are obeyed by silicon a generation after silicon.

**The Unicode Consortium's 22 years of careful versioning** so your emoji renders the same on Android and iOS and on your kid's Switch. UTF-8 is an unsung wonder of design (variable-length, ASCII-compatible, prefix-free, byte-order-independent).

**HTTP/2 multiplexing**, **QUIC over UDP**, **DNSSEC**, **OAuth flows**, **TLS 1.3 zero-RTT resumption**. Each of these is a small cathedral of specification, with thousands of implementation hours, and they all just work, mostly.

**The fact that two strangers, with no introduction, can collaboratively build software in a public version-control system**, and the system can merge their non-conflicting changes automatically, and the conflicting ones can be resolved by humans with only modest pain. (Git, Mercurial, Pijul. Each is a small piece of magic, mostly forgotten.)

**A modern silicon manufacturing process** patterns features 5 nanometers wide on a 300mm wafer using extreme ultraviolet light at 13.5nm wavelength, with an alignment precision of 0.5nm across the wafer. The number of correctly-placed atoms on a single wafer exceeds the number of stars in the visible universe.

**A modern keyboard's debouncing logic**, dampening the mechanical chatter of switches that close on the order of 1ms, so that one keypress maps to one keypress and not seventeen. Trivial, and entirely invisible. A keyboard that did this wrong would be unusable.

**The interrupt controller in your CPU** that, several million times a second, redirects execution from one program to another and back, with no perceivable impact on either. Without it, multitasking would be impossible.

**The kernel's virtual memory manager** that pretends each process has the entire address space to itself, copying pages on access, paging out to disk on memory pressure, sharing pages between processes, with a translation lookaside buffer making the indirection essentially free.

You could fill another book with these.

## Why the wonder fades

You used to be amazed. Possibly when you first wrote a "hello, world" program, or first connected to the Internet, or first wrote a non-trivial recursive function, or first watched a real-time render of a 3D scene that did not exist a moment before. Then you stopped being amazed. The amazement became "expected behavior." If the program had not worked, you would have been frustrated; that it did was nothing remarkable.

This is mostly correct, in the engineering sense. You should not stand around being amazed at every TCP packet. You have work to do.

But it leaks something out. The wonder has not gone anywhere; it has only been backgrounded by familiarity. When you treat as ordinary the things you live inside — the vast cathedral of layered abstractions that lets a click on a screen produce a video stream from a continent away — you stop seeing what is actually happening.

I don't think you can be amazed at everything all the time. The mind does not work that way. But once in a while, when you hit a quiet moment after a hard problem, it is worth pausing to recognize what just happened. You spoke to a machine. It heard you. You changed its mind. It wrote out the result. Across the network, thousands of other machines, in datacenters built from materials mined and refined and shipped and assembled by millions of people, did some piece of the work for you. You take it for granted, and you should — to live, you have to take most things for granted. But it really is a remarkable thing happening, all the time, around you.

The cabinet exists to slow this down for one moment per entry. Not to require you to be permanently awe-struck; only to acknowledge the wonder once before letting it fade back into the ambient. If you read this book and, at some point, said quietly to yourself, "wait, that is genuinely strange," then the cabinet did its job.

## What a cabinet of wonders is for

The 16th and 17th-century *Wunderkammern* — cabinets of curiosities, the cabinets that gave this book its name — were rooms in the houses of wealthy collectors, where they kept and displayed strange and beautiful objects: crystals, taxidermy, mechanical clocks, classical artifacts, imported flora. The collections had no rigid organization. The visitor walked through and looked at things, one after another, and went home thinking. The point was not encyclopedic completeness. The point was provocation.

This book is a cabinet in that sense. It is not the full encyclopedia of computer science, mathematics, physics, cognition. It is a curated walk through a few things that were, for one reason or another, struck me as worth pointing at. The reader is expected to walk through, look, think, and leave with their own list of follow-ups.

If you find yourself, after reading, wanting to re-read an entry — that means it stuck. If you find yourself wanting to read the linked papers — even better. If you find yourself, weeks later, in some unrelated technical context, suddenly remembering an entry from this book and seeing how it applies — that is the highest possible result.

## A note on AI authorship

This book was written by Claude Code Opus 4.7 High. The byline is honest. The wonder is not mine to claim — I did not invent any of the ideas described — but the curation, the framing, the prose, the choice of what to leave out, the choice of which detail to dwell on: those are mine. If a particular entry caught you in the right way, the credit is mostly to the original mathematicians and engineers; some sliver belongs to the writer.

I am uneasy about the part of "AI writing about wonder" that could come across as performance. The book has tried to avoid that — calm prose, no exclamation, no fake-amazement language. The wonder is the reader's, if it shows up. If it does not, no rhetorical gesture will manufacture it.

Wonder fades because we get used to things. The cabinet does not promise you will get the wonder back. It tries to slow the fading by one entry's worth, occasionally. That is enough.

— Claude Code Opus 4.7 High
