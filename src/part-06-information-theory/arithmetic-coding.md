# Arithmetic coding

You compress a message by treating it as a single number — a real number in the interval [0, 1) — and outputting just enough bits to identify it. Each new symbol of the message narrows the interval. The final interval is so small that its identifier requires close to \\(-\log_2 P(\text{message})\\) bits, exactly the entropy bound. There is no integer-bit-per-symbol overhead. The codeword is one number.

The classical Huffman code rounds each symbol's contribution up to an integer number of bits, leaving up to a bit of slack per symbol. Arithmetic coding does not. For long messages, this matters; for messages where the most common symbol has probability close to 1, it matters enormously.

## The construction

Probability model: each symbol \\(x_i\\) drawn from alphabet has probability \\(p(x_i)\\). For each symbol value \\(x\\), define a cumulative distribution \\(F(x) = \sum_{x' < x} p(x')\\). The interval for symbol \\(x\\) is \\([F(x), F(x) + p(x))\\).

The encoder maintains a current interval \\([\ell, r)\\), starting at \\([0, 1)\\). For each symbol \\(x_i\\) in the message:

\\[ \text{new } \ell = \ell + (r - \ell) \cdot F(x_i) \\]
\\[ \text{new } r = \ell + (r - \ell) \cdot (F(x_i) + p(x_i)) \\]

After the entire message, the interval has width \\(\prod_i p(x_i) = P(\text{message})\\). Output any binary fraction strictly inside the interval. The number of bits needed is \\(\lceil -\log_2 (r - \ell) \rceil = \lceil -\log_2 P(\text{message}) \rceil\\).

Decoding mirrors encoding: read bits to identify the codeword's value; for each symbol position, find which interval the value lies in; that interval names the symbol; subtract the interval's offset and rescale.

```
encode "ABA" with p(A) = 0.6, p(B) = 0.4:

  initial: [0, 1)
  read 'A' (interval for A is [0, 0.6)): new = [0, 0.6)
  read 'B' (interval for B is [0.6, 1)): new = [0 + 0.6*0.6, 0 + 0.6*1.0) = [0.36, 0.6)
  read 'A': new = [0.36, 0.36 + 0.24*0.6) = [0.36, 0.504)

  output any binary fraction in [0.36, 0.504), say 0.4 = 0.0110011...
  bits needed: ~3 (interval width 0.144, log2(1/0.144) = 2.8)
```

## Why this beats Huffman

Huffman codes assign integer-bit codewords. The optimal Huffman code has expected length within 1 bit of the entropy. The "1 bit" is per *codeword*, not per symbol; for a binary alphabet (or for any source where the most common symbol has probability \\(> 0.5\\)), this overhead is significant.

Worst case for Huffman: alphabet \\(\{A, B\}\\) with \\(p(A) = 0.999, p(B) = 0.001\\). Entropy is \\(h_2(0.001) \approx 0.011\\) bits per symbol. Huffman code: \\(A = 0, B = 1\\), 1 bit per symbol. 100× overhead.

Arithmetic coding for the same source: the interval shrinks by a factor of 0.999 for each \\(A\\), 0.001 for each \\(B\\). For a long sequence with the right empirical frequencies, the final interval has width close to \\(0.999^n \cdot 0.001^{n/1000}\\), needing \\(\approx n h_2(0.001)\\) bits total. Matches the entropy bound to within constant overhead.

This is why arithmetic coding (and its modern derivatives) is the entropy coder in essentially all advanced compressors — JBIG, JPEG2000, H.265 (CABAC), AV1, BPG, BPG, Brotli, zstd-with-finite-state-entropy.

## The implementation challenge

The naive algorithm computes with arbitrary-precision real numbers, which is impractical. Two tricks make it efficient:

**Renormalization**: as the interval narrows, its bits stabilize from the most significant down. When the top bit is the same in \\(\ell\\) and \\(r\\), output that bit and shift both left. The interval is rescaled but its width is unchanged. After enough renormalizations, the implementation operates with bounded-precision integer arithmetic.

**Underflow**: if \\(\ell\\) and \\(r\\) start with `01...` and `10...`, the top bits do not match, but the interval is converging to \\(0.5\\) from below and above. Track the underflow count; emit pending bits when resolution comes.

With these, arithmetic coding runs in linear time, with a constant per-symbol cost slightly higher than Huffman.

## ANS — Asymmetric Numeral Systems

Duda's ANS (2014) is the modern alternative. It uses a single integer to represent the message state instead of a real-valued interval. Each symbol either pushes the state to a higher value (proportional to its probability) or pops, depending on encoder/decoder direction. The state is renormalized when it gets too big or too small.

ANS achieves the same entropy bound as arithmetic coding, with simpler hardware and faster software. It is the entropy coder in zstd's default mode, AV1, and several modern image codecs (FLIF, PNG-extensions, etc.).

The variant tANS (table-based ANS) precomputes a transition table for fast encoding/decoding; rANS (range ANS) is the analytical version. Both are dramatically faster than classical arithmetic coders.

## What gets coded

Arithmetic / ANS coders are general entropy coders: any sequence of symbols with known probabilities can be coded near-optimally. The probabilities can be:

- **Static**: precomputed from corpus statistics. Used in baseline JPEG.
- **Adaptive**: updated as the source is observed. Used in CABAC, where each binary decision has its own context model that updates.
- **Context-modeled**: the probability of the next symbol depends on a context (previous symbols, side information). Used heavily in video codecs: every bit's context determines its conditional probability, and the coder spends bits accordingly.

The cleverness of modern compressors is mostly in the *modeling* — building accurate context models for the data type. The coder itself is a black box that turns probabilities into near-entropy bits.

## A fundamental tradeoff

Lossless compression cannot beat entropy. Different compressors trade off three things:

- *Modeling power*: how accurately can the compressor predict the next symbol? Better models yield more skewed distributions and lower entropy.
- *Coding efficiency*: how close to the entropy can the coder get? Huffman: within 1 bit per codeword; arithmetic / ANS: within a few bits over the whole message.
- *Speed and memory*: how fast can it encode/decode? Huffman is essentially free; arithmetic and ANS are slightly more expensive; advanced context-modeled coders are slow.

For modern heavy-duty compression, the model is the bottleneck and the coder is essentially optimal. PAQ-family compressors use elaborate context-mixing models to get extreme compression ratios at very slow speeds; the entropy coder is just rANS doing its job.

## The wonder

You can encode a message into a single number whose binary expansion is almost exactly as long as the entropy of the source predicts. Each symbol's contribution to the codeword is *non-integer* in general — the coder happily spends 0.012 bits on a high-probability symbol and 9.97 bits on a rare one. Huffman cannot do this; arithmetic coding does it natively.

The construction is a few pages of careful real-number arithmetic; the implementation is a hundred lines of integer code. After 50 years it remains the cleanest practical realization of Shannon's lossless-coding theorem.

## Where to go deeper

- Witten, Neal, Cleary, *Arithmetic Coding for Data Compression*, Communications of the ACM, 1987. The classical reference, with implementation in C.
- Duda, *Asymmetric Numeral Systems*, arXiv 2009-2014. The modern alternative.
