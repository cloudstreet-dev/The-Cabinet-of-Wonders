# Network Time Protocol

Your laptop's clock agrees with the clock on a server eight thousand kilometers away to within a few milliseconds, in spite of variable network latency, asymmetric routing, packet jitter, and the fact that neither machine has any direct way to observe the other's idea of "now." It does this with four timestamps and a tiny piece of arithmetic. The protocol is older than the World Wide Web. It still works.

## Why it should not work

The network gives you a one-way trip whose duration you cannot measure. A round-trip is observable — send a packet, time the response — but a one-way trip is not, because to time a one-way trip you would need synchronized clocks at both ends, which is the problem you are trying to solve.

If you assume the trip is symmetric — out and back take equal time — you can divide the round-trip in half and call it the one-way time. That assumption is wrong on the modern Internet, often badly so (asymmetric paths through different ASes routinely have 5–50 ms differences). NTP knows this and disclaims accuracy claims tighter than the path asymmetry. Within that limit, it works extraordinarily well.

## The four timestamps

Client and server exchange one packet pair. Each side records two timestamps:

```
Client clock     Server clock
   T1  ----------->
                       T2 (server records receipt)
                       T3 (server sends response)
   T4  <-----------
```

T1 and T4 are read from the client's clock. T2 and T3 are read from the server's clock and travel inside the response packet. After the exchange, the client has all four numbers.

Two quantities follow from them:

\\[ \text{round-trip delay: } \delta = (T_4 - T_1) - (T_3 - T_2) \\]

\\[ \text{clock offset: } \theta = \frac{(T_2 - T_1) + (T_3 - T_4)}{2} \\]

The delay is straightforward: total elapsed time on the client side, minus the time the server held the packet. Subtraction cancels the server's clock offset (T3 - T2 is purely on the server's clock; T4 - T1 is purely on the client's; they share no terms).

The offset estimate is more interesting. Let \\(\theta\\) be the true offset (server time minus client time at the same instant), and let \\(d_1, d_2\\) be the actual one-way delays out and back. Then:

\\[ T_2 = T_1 + d_1 + \theta \quad \implies \quad T_2 - T_1 = d_1 + \theta \\]
\\[ T_4 = T_3 + d_2 - \theta \quad \implies \quad T_3 - T_4 = -d_2 + \theta \\]

Add and divide by 2:

\\[ \frac{(T_2 - T_1) + (T_3 - T_4)}{2} = \theta + \frac{d_1 - d_2}{2} \\]

If \\(d_1 = d_2\\), the right side is exactly \\(\theta\\). If they differ, the offset estimate is wrong by half the asymmetry. There is no way to do better with these four numbers; you cannot recover \\(\theta\\) and \\(d_1\\) and \\(d_2\\) separately because the system has only two equations.

So NTP's accuracy floor is the path asymmetry. On a well-behaved LAN, sub-millisecond. On a long-haul Internet path, a few milliseconds. On a satellite link or a path with serious queueing on one side, much worse, and NTP knows to advertise wide error bounds in those conditions.

## The hierarchy

A planet's worth of clocks cannot all peer with each other. NTP organizes servers in strata:

- **Stratum 0**: physical clock sources — atomic clocks, GPS receivers, radio time signals (DCF77, WWVB). Not on the network.
- **Stratum 1**: servers directly attached to a stratum-0 reference. They are the timekeepers of the Internet, of which there are several thousand worldwide.
- **Stratum 2**: servers that synchronize from stratum-1 servers.
- **Stratum 3+**: each level synchronizes from the level above.

Your laptop is typically stratum 3 or 4, which is plenty. The accuracy degrades by the path-asymmetry term at each hop, but the degradation is small relative to the wall-clock precision most applications care about.

## Filtering, smoothing, slewing

A single offset measurement is too noisy to act on. NTP keeps a window of recent measurements, picks the eight with the smallest delay (low delay correlates with low queueing, which correlates with low asymmetry), then computes a weighted average. From multiple servers, it runs a Marzullo-style intersection algorithm to find the offset region consistent with the largest group of "truechimers" and discards the disagreeing minority — a Byzantine agreement step that handles a malicious or broken time source without amplifying its error.

Once an offset is decided, the client does not just slam its clock to the new value. It would break monotonicity (programs assume time only moves forward) and invalidate timestamps in flight. Instead, the client *slews* the clock: speeds it up or slows it down by a small fraction (max 500 ppm in standard implementations) until the offset closes. Big jumps happen only at boot, when nothing is yet relying on the clock.

The same loop also estimates the local clock's *frequency error* — every quartz oscillator drifts at some constant rate (parts per million) plus temperature-dependent variations — and the Linux kernel keeps a frequency adjustment register that compensates. After a few hours of NTP, your machine's clock is keeping time to within a few parts per million on its own, and NTP only has to nudge for path-asymmetry-level corrections.

## What it costs

The protocol itself is one UDP packet of 48 bytes in each direction, exchanged every 64 to 1024 seconds. Steady-state, NTP costs about a packet every few minutes. A stratum-1 server can serve millions of clients on a single CPU. The pool.ntp.org cluster, run by volunteers, serves most of the consumer Internet.

For applications that need tighter sync — high-frequency trading, distributed databases, telecoms — there is PTP (Precision Time Protocol, IEEE 1588), which moves the timestamping into the network hardware to remove software-stack jitter, and which can deliver sub-microsecond sync on a switched LAN. The conceptual move is the same: hardware-stamped timestamps in both directions, careful filtering, frequency tracking. The math of inferring offset from four timestamps is identical.

## The wonder

A consumer machine, listening to no master clock, on a network where one-way delay cannot be measured, agrees with the rest of the world to a few milliseconds, by sending a single packet pair every couple of minutes and doing fifth-grade arithmetic on the result. The whole edifice runs on a UDP service that you have probably never thought about. Without it, every certificate would be wrong, every distributed database would split-brain, every replicated log would lose causality, and every cron job would run at the wrong moment. With it, time is a global free service nobody bills for.

## Where to go deeper

- David Mills, *Computer Network Time Synchronization* (CRC Press, 2010). The book by the protocol's author. Idiosyncratic, comprehensive.
- RFC 5905 (NTPv4). The current spec. Read alongside the book; the RFC alone is dense.
