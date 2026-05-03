# Network coding

If you have a network of pipes carrying data and you let the routers along the way do something more than relay packets — specifically, let them XOR or otherwise mix incoming packets to produce outgoing ones — you can sometimes achieve throughput that no routing strategy can match. The classical view that routers are just forwarding switches is, in some topologies, off by a factor of \\(\log n\\) or more.

Ahlswede, Cai, Li, and Yeung published this in 2000. It overturned a decades-old assumption that "routing is the right abstraction for networks." It also unlocked some clever applications in distributed storage, peer-to-peer streaming, and wireless multicasting.

## The classical "butterfly" example

Two sources \\(s_1, s_2\\) want to send their messages \\(b_1, b_2\\) to two sinks \\(t_1, t_2\\). The network has unit-capacity links arranged like this:

```
              s_1            s_2
             /   \\           /   \\
            /     \\         /     \\
           v       \\       /       v
          A         \\     /         B
          |          \\   /          |
          |           \\ /           |
          |           v v            |
          |            R             |
          |            |             |
          |            v             |
          |            S             |
          |           / \\            |
          |          /   \\           |
          v         v     v          v
         t_1                        t_2
       (wants                      (wants
       both)                       both)
```

Each link carries one bit per time slot. \\(s_1\\) needs to deliver \\(b_1\\) to both sinks; \\(s_2\\) needs to deliver \\(b_2\\) to both sinks. The min-cut from \\(\{s_1, s_2\}\\) to \\(\{t_1, t_2\}\\) has capacity 2 in each direction, so a multicast rate of 2 should be achievable.

If routers can only relay, the bottleneck link \\(R \to S\\) can carry only one of \\(b_1, b_2\\). Whichever one is routed through it, the other sink misses one bit per round. Throughput is bounded above by 1.5.

If \\(R\\) can XOR, the bottleneck carries \\(b_1 \oplus b_2\\). The downstream node \\(S\\) sends this XOR to both sinks. Sink \\(t_1\\) already has \\(b_1\\) (from a separate path through \\(A\\)) and computes \\(b_2 = b_1 \oplus (b_1 \oplus b_2)\\). Sink \\(t_2\\) does the dual computation. Both sinks recover both messages. Throughput: 2 per round.

The XOR is the wonder. In the relay model the bottleneck cannot carry two messages; in the coding model it carries a *combination* that decomposes back into both.

## The general theorem

**Multicast capacity theorem** (Ahlswede et al., 2000): in a directed acyclic network with one source and \\(t\\) sinks, all wanting the same message stream, the maximum achievable rate equals the minimum, over all sinks, of the max-flow from source to that sink.

Equivalently, you can achieve the multicast capacity. This was *not* true under pure routing.

For unicast (one source, one sink), routing already achieves the max-flow capacity (Ford-Fulkerson). The win for network coding is in *multicast* and *multiple-unicast* (multiple source-sink pairs sharing a network).

## Linear network coding

Li, Yeung, Cai (2003): for multicast, *linear* network coding suffices. Each intermediate node computes its outputs as linear combinations (over some finite field) of its inputs. Sinks solve a system of linear equations to recover the source message.

Specifically, each packet on each link carries a vector of source bits *plus* a coefficient vector indicating which linear combination it represents. Sinks collect enough coefficient-tagged packets to invert the matrix and recover the original messages.

For a max-flow of \\(h\\), the field size needs to be at least the number of sinks (or so). For a typical multicast scenario, GF(\\(2^8\\)) or GF(\\(2^{16}\\)) suffices.

## Random linear network coding

Ho, Médard, Koetter, Karger, Effros (2006): even *random* linear combinations work, with high probability. Each intermediate node picks random coefficients in some finite field, mixes its incoming packets accordingly, and forwards. Sinks decode if and only if they receive a full-rank set of mixed packets, which they do with probability close to 1 for large enough fields.

Random linear network coding is the dominant practical version. It does not require centralized topology knowledge or scheduled coding decisions. Each node operates locally with random combinations, and the system achieves capacity on average.

## Where it matters

**Distributed storage with regenerating codes**: when a node fails in a distributed-storage system, the system must read from \\(k\\) other nodes and reconstruct the lost data. With Reed-Solomon codes, this requires reading the full \\(k\\) blocks. With network-coding-based regenerating codes, intermediate nodes can mix data so that recovery requires less network bandwidth — sometimes as little as the size of the lost block plus a small overhead.

**Wireless mesh networks**: in a mesh, a single broadcast from one node can be heard by multiple neighbors. Network coding lets the broadcast carry the XOR of multiple intended packets; each receiver decodes its own using already-known packets. Reduces airtime substantially in shared channels.

**Coded TCP and erasure-coded streaming**: in lossy networks, network coding lets sources send linear combinations of packets. Receivers need any \\(k\\) of \\(n\\) packets, in any order, to recover the original \\(k\\) packets. Resilient to packet loss without retransmission. Used in some IoT and streaming-video protocols.

**P2P streaming**: Avalanche (and successors): each peer mixes the chunks it has and forwards to others. Receivers need any \\(k\\) coded chunks (out of more than \\(k\\) circulating) to reconstruct. Fixes the "rare-chunk problem" in BitTorrent-style protocols.

## What it does not do for the open Internet

The Internet's routers do not perform network coding. Adoption has been slow because:

- **Layering**: network coding requires intermediate nodes to be aware of *which* packets to combine. The IP layer is intentionally dumb — routers do not know which packets are part of the same flow.
- **Encryption**: end-to-end encrypted packets cannot be linearly combined by intermediate routers without breaking encryption.
- **Sufficient bandwidth**: for unicast traffic, classical routing already achieves capacity. The advantages of network coding kick in for multicast or multiple-unicast, both of which are smaller use cases on the open Internet.

It does show up in narrow domains: distributed storage (Coded Cache, regenerating codes in Backblaze and others), wireless mesh routing, content distribution networks at the edge.

## Why this is conceptually wonder

The pre-2000 model: a network is a graph; capacity is max-flow; routing implements it. This had been the foundation of network theory since Ford-Fulkerson in 1956.

Network coding pointed out that the model was over-constrained. The graph metaphor made each link a pipe carrying packets, and each node a switch directing them — passive elements moving the source's bits unchanged. But there is no theoretical reason intermediate nodes cannot *operate* on the packets. Once you let them, the achievable rate region opens up.

The multicast capacity theorem is one of those results that breaks an unstated assumption everyone had been making. The assumption was: information is conserved through a network like water through pipes; the routers carry it but do not transform it. Network coding showed: that is just one strategy. Allow transformation, and the same network can carry strictly more. The graph topology is the same; the packets are the same; only the operation at the routers is more clever, and the throughput goes up.

For the right kind of structured-traffic workloads (multicast, distributed storage, wireless), the gain is real and quantifiable. For unstructured unicast on the open Internet, less so. But the theoretical contribution stands: information networks are not pipe networks. They are computation networks whose nodes can do more than relay.

## Where to go deeper

- Ahlswede, Cai, Li, Yeung, *Network Information Flow*, IEEE Transactions on Information Theory, 2000. The defining paper.
- Yeung, *Information Theory and Network Coding* (2008). Modern textbook treatment.
