# TCP congestion control

A few billion machines, owned by people who have never spoken to each other, share a network with no central scheduler, no admission control, and no mandatory rate limits. They each get a fair slice of the available bandwidth, the network avoids melting under load, and the whole thing is held together by a feedback loop that runs entirely on the endpoints. The routers in the middle do not even know what TCP is.

## What goes wrong without it

Imagine two computers connected by a link of capacity \\(C\\). The sender wants to push as fast as possible. If it sends faster than \\(C\\), packets queue at the bottleneck router. The queue fills, then overflows. Overflowed packets are dropped. Higher-layer protocols retransmit. The retransmissions add to the load. Queueing delay rises until round-trip time is dominated by queue length, and the network's effective throughput collapses.

This is the congestion collapse the early Internet actually suffered. In October 1986 the throughput between LBL and UC Berkeley, separated by 400 yards of fiber, dropped from 32 kbps to 40 bps — three orders of magnitude. Van Jacobson's response, published in 1988, is the algorithm we still use, with refinements. It changed nothing about the routers. It only changed how endpoints decide when to slow down.

## The core idea

Each TCP sender maintains a number called the congestion window, `cwnd`, in bytes (or packets). The sender is allowed to have at most `cwnd` bytes in flight — sent but not yet acknowledged. If the round-trip time is `RTT`, then sending rate is approximately `cwnd / RTT`.

The algorithm probes for capacity. It increases `cwnd` until something goes wrong, then backs off. The shape of the increase and decrease is the entire question.

The classical Reno algorithm:

- **Slow start.** When a connection opens, `cwnd` starts at 1 (later, 10) packets and doubles every RTT. Specifically, each ACK increases `cwnd` by one packet, so a windowful of ACKs roughly doubles `cwnd`. This continues until either a loss occurs or `cwnd` reaches a slow-start threshold `ssthresh`.
- **Congestion avoidance.** Once `cwnd >= ssthresh`, the sender increases `cwnd` by 1/cwnd per ACK, so it gains roughly one packet per RTT. Linear, not exponential.
- **Loss as signal.** When a packet is lost (detected by three duplicate ACKs or a timeout), the sender concludes the network is congested. It halves `cwnd` and `ssthresh`, and resumes from there.

This is the AIMD pattern: Additive Increase, Multiplicative Decrease. The sender adds a constant per RTT, and on loss it cuts by a constant fraction.

```
cwnd
 |
 |        /\        /\        /\
 |       /  \      /  \      /  \
 |      /    \    /    \    /    \
 |     /      \  /      \  /      \
 |    /        \/        \/        \
 |   /
 |  /
 | /
 |/_______________________________________ time
   slow start    AIMD sawtooth
```

## Why AIMD gives fairness

Two flows share a link. Both run AIMD. Plot `cwnd_1` on x-axis, `cwnd_2` on y-axis. Each flow's state is a point in the plane.

Both flows are increasing additively most of the time, so the point moves at 45° toward the upper right. When the link saturates and packets drop, both flows experience loss and halve their windows, so the point jumps toward the origin along a line through the origin.

The line "fair share" is \\(\text{cwnd}_1 = \text{cwnd}_2\\). Additive moves are parallel to that line. Multiplicative moves are toward the origin, which preserves the ratio \\(\text{cwnd}_1 / \text{cwnd}_2\\) — but it also reduces the absolute distance from the fair-share line, because halving brings the point closer to the origin, where the fair-share line and any ray from the origin both pass through.

```
cwnd_2
  ^
  |        /  fair share line
  |       /
  |     A/  <- start here
  |     /\
  |    /  \  additive: parallel to fair share
  |   /    \
  |  /     B  <- multiplicative: toward origin,
  | /     .     ratio preserved, distance to
  |/    .       fair-share line shrinks
  |   .
  | .
  +-----------------------> cwnd_1
```

Iterate this dance and the point converges to the fair-share diagonal. Two flows running AIMD converge to equal share without any signaling, without identifying each other, without trusting each other. The geometry forces fairness.

Multiplicative-increase, multiplicative-decrease (MIMD) keeps you on the diagonal but oscillates on the same ray, which is unstable. AIAD never recovers ratio after a perturbation. AIMD is the only one of the four options that converges to fair from any starting point. There is a real theorem here (Chiu and Jain, 1989), and the geometry above is the proof, drawn.

## What loss-based AIMD pays for it

The pure Reno picture has a flaw: it requires loss as the signal. To probe for capacity, the sender must overflow the queue, drop a packet, retransmit, and back off. The bottleneck queue is therefore deliberately filled to overflowing, all the time. This is why your Wi-Fi feels laggy when you are saturating it: the bottleneck queue is full and your latency is dominated by queueing delay.

Modern algorithms try to escape this:

- **CUBIC** (Linux default since 2.6.19, ~2006) replaces the linear AIMD increase with a cubic function of time since last loss. Near the previous saturation point, `cwnd` increases slowly; far from it, fast. This handles high-bandwidth long-RTT networks better, where the time to grow `cwnd` linearly back to capacity would be ridiculous.
- **BBR** (Google, 2016) abandons loss as the signal entirely. It models the bottleneck as a pipe with a bandwidth-delay product, periodically estimates the bottleneck bandwidth and minimum RTT, and paces sends to match. It deliberately avoids queue buildup, achieving high throughput with low queueing delay. It is dramatically faster on lossy long-haul links, where loss is not a congestion signal but a transmission error.

## What the routers are doing

Almost nothing. A standard router queues packets in a single FIFO and drops the tail when the queue fills. This is "Drop Tail." It works because the endpoints react. Routers can do better — Random Early Detection drops a probabilistic fraction of packets as the queue grows, signaling congestion before the queue overflows. ECN (Explicit Congestion Notification) does the same thing without dropping: a router marks a bit in the IP header to say "I am congested," and the receiver echoes it to the sender, which reacts as if it had seen a loss. Both shave off the worst latency tails of pure Drop Tail.

But fundamentally, the system is endpoint-driven. The protocol runs on your laptop and on the server. It does not require any router on the path to know its name. That is why TCP rolled out in the 1980s and still works today, despite the routers, links, and traffic having changed by many orders of magnitude.

## The wonder

A control system this large, this distributed, with this many adversarial actors, and no central authority, should not work. There is no committee deciding how much bandwidth Netflix gets versus a video call. There is no admission control deciding whether you may begin a TCP session. The mechanism is just: every endpoint runs roughly-the-same algorithm, in software it can change at any time, and the algorithm is designed so that the geometry of the strategy space pulls everyone toward fairness. It works because the math says it has to.

## Where to go deeper

- Van Jacobson, *Congestion Avoidance and Control*, SIGCOMM 1988. The original. Twenty pages, very readable.
- Cardwell, Cheng, Gunn, Yeganeh, Jacobson, *BBR: Congestion-Based Congestion Control*, ACM Queue 2016. The explicit break with loss-based control.
