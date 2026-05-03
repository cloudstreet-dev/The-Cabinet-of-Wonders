# LDPC codes

Shannon's noisy-channel theorem says that for any channel with capacity \\(C\\), error-correcting codes can transmit at any rate below \\(C\\) with arbitrarily small error probability. The theorem is from 1948. For 45 years, no one knew a code with practical decoders that came within sight of the Shannon bound. Standard codes operated 3–5 dB short of capacity — meaning real systems needed several times more transmitted power than the theory said was minimum.

Then in 1993, two events. First, Berrou-Glavieux-Thitimajshima introduced *turbo codes*, which got within 0.5 dB of Shannon. Second, MacKay rediscovered Gallager's 1962 PhD thesis on *low-density parity-check (LDPC) codes*, which had been forgotten for 30 years, and showed they did the same thing — at lower decoding cost.

Today every modern wireless standard, every modern wired standard, and the storage layer of every modern SSD uses LDPC or turbo codes. They are the closest thing engineering has come to operating literally at the Shannon limit.

## The construction

An LDPC code is defined by a *parity-check matrix* \\(H\\) of dimensions \\((n - k) \times n\\). A codeword \\(c \in \mathbb{F}_2^n\\) is any binary vector satisfying

\\[ Hc = 0 \\]

over \\(\mathbb{F}_2\\). The codewords form a linear subspace of dimension \\(k\\), so there are \\(2^k\\) of them. The rate is \\(k/n\\).

The "low-density" part: \\(H\\) has very few 1s — a sparse matrix, with each row containing only \\(d_c\\) ones (column weight) and each column containing only \\(d_v\\) ones (variable weight), with \\(d_c, d_v\\) constants independent of \\(n\\). So \\(H\\) is mostly zeros, with a sparse pattern of constraints.

This sparsity is everything. It enables fast decoding, and it gives the code its near-Shannon-limit performance.

## The Tanner graph

Visualize \\(H\\) as a *Tanner graph* — a bipartite graph with \\(n\\) variable nodes (one per codeword bit) and \\(n - k\\) check nodes (one per parity check), with edges where \\(H_{ji} = 1\\). Each check node connects to \\(d_c\\) variable nodes (the bits whose XOR must equal 0). Each variable node participates in \\(d_v\\) checks.

```
   variable nodes (codeword bits)
     |     |     |     |     |
     o     o     o     o     o
    / \\   /|\\   /|     |\\   |
   /   \\ / | \\ / |     | \\  |
  +     X   X    +      +    +
  |   /  \\  |    \\    /     |
  +--+    +-+     +--+-+      +
     check nodes (parity equations: XOR = 0)
```

This graph has rules: each check node says "the XOR of my variable neighbors is 0." Decoding is "given noisy observations of the variables, find an assignment that satisfies all the checks."

## Belief propagation decoding

The decoder is *iterative message-passing* on the Tanner graph:

1. Initialize each variable node with a probability (or log-likelihood ratio, LLR) reflecting the channel observation.
2. Variable nodes send their current belief to each check node.
3. Each check node receives beliefs from its variable neighbors, computes — for each neighbor — what the constraint says the neighbor's bit should be (given the others). Sends this back as a belief.
4. Each variable node combines its channel observation with the beliefs from all its check neighbors. Updates its belief.
5. Iterate.

After enough iterations (typically 10-50), the beliefs converge to a confident assignment that satisfies all the checks (with high probability, if the channel was within the code's threshold).

Each iteration costs \\(O(n)\\) operations because of the sparsity. Total decoding cost is \\(O(n)\\) per iteration times constant iterations: linear in \\(n\\). This is what made LDPC practical.

## Why it works near Shannon

Belief propagation is exact on trees. The Tanner graph of a sparse LDPC code is, locally, tree-like — short cycles are rare in random sparse bipartite graphs. So belief propagation is *almost* exact, and converges to a good answer for codes designed to have few short cycles.

The threshold phenomenon: for each LDPC code family, there is a critical channel SNR (or noise level) below which BP-decoding succeeds with probability approaching 1 as \\(n \to \infty\\), and above which it fails. The threshold can be computed analytically by *density evolution* — tracking the distribution of message values through iterations, in the large-\\(n\\) limit.

For well-designed irregular LDPC codes (variable-degree distributions optimized for the channel), the threshold can be made arbitrarily close to the Shannon limit. Gallager's original regular codes were 0.5–1 dB short; modern irregular codes are within hundredths of a dB.

## Engineering details

**Quasi-cyclic LDPC**: real codes have structured \\(H\\) matrices made of permuted identity blocks, allowing efficient hardware implementation. WiFi (802.11n/ac/ax/be), 5G NR, DVB-S2, all use QC-LDPC.

**Encoding**: a sparse \\(H\\) does not directly give a sparse encoder. Modern codes either accept \\(O(n^2)\\) encoding (small \\(n\\)), use approximate triangular structure for \\(O(n)\\) encoding, or design the code to be systematic with a structured generator.

**Erasure decoding**: on the binary erasure channel (each bit either received correctly or marked erased), LDPC decoding is just iterative substitution: any check node with one unknown variable propagates the value. This is the *peeling decoder* and is the basis for fountain codes (Raptor, LT, Online).

## Where they show up

- **Wi-Fi**: 802.11n introduced LDPC as an option; 802.11ac and later use it heavily.
- **5G New Radio**: LDPC is the data channel code (Polar codes are used for the control channel — see below).
- **DVB-S2 satellite TV**: LDPC + outer BCH.
- **10GBASE-T Ethernet**: LDPC for noise margin on copper.
- **Storage**: SSDs use LDPC over flash NAND cells, which have raw bit error rates in the percent range. Without LDPC, modern multi-level flash would be useless.
- **Hard drives**: also LDPC.
- **Optical comms**: long-haul fiber typically uses LDPC concatenated with Reed-Solomon.

## Polar codes

A close cousin: *polar codes* (Arıkan, 2008) achieve channel capacity for binary input symmetric memoryless channels with provably optimal asymptotic performance. They are used in 5G's control channel and are theoretically beautiful. LDPC remains the workhorse for data channels because of its lower complexity at finite block lengths.

## Why this is a wonder

The Shannon limit, set in 1948, was a hard upper bound on what error correction could possibly do. For 45 years, the gap between theory and practice was several dB — meaning real systems were operating with several times more redundancy than the theory said was strictly necessary. The story was that the theorem was non-constructive: it said codes existed but did not give them.

LDPC closed the gap. The gap that engineers struggled with for decades collapsed once they tried sparse parity-check matrices with iterative decoding — a construction Gallager had proposed in 1962 but that everyone abandoned because the matrix was too big to invert directly. Iterative decoding does not invert the matrix; it propagates beliefs along edges. Linear time, near-optimal performance, on what had been a famously hard frontier.

The construction's reach is huge: every TV signal, every mobile data session, every flash chip, every DSL line, every satellite downlink. The sparse parity-check matrix is doing the work in all of them.

## Where to go deeper

- Gallager, *Low-Density Parity-Check Codes*, MIT Press, 1963 (his thesis). The original.
- MacKay, *Information Theory, Inference, and Learning Algorithms*, Chapters 47–50. Modern, free online, and beautifully written.
- Richardson and Urbanke, *Modern Coding Theory*. The reference for the theory of capacity-approaching codes.
