# The source coding theorem

A long English text can be compressed to about 1.5 bits per character — a third of what an ASCII representation uses. Compression any tighter than that, on average, is provably impossible. The bound is exact, the proof is two pages, and the same theorem applies to any random source: there is a number, the entropy, below which no compression scheme can go, and arbitrarily close to which a sufficiently clever scheme can come.

Shannon proved this in 1948. It is the founding theorem of information theory. Every compressor — gzip, zstd, JPEG, video codecs, MP3 — operates inside the bound it set down.

## The setup

A *source* is a probability distribution \\(p\\) over a finite alphabet. A long message is a sequence of symbols drawn i.i.d. from \\(p\\). A *code* is a way of representing each symbol (or each block of symbols) as a binary string. We want the codes to be *uniquely decodable*: any concatenation of codewords decodes uniquely to the original sequence.

A code's *expected length* is \\(\sum_x p(x) \cdot L(x)\\) where \\(L(x)\\) is the length of \\(x\\)'s codeword. We want to minimize this.

## The entropy

For a discrete distribution \\(p\\), the *entropy* is

\\[ H(p) = -\sum_x p(x) \log_2 p(x) \\]

measured in bits. It is a single number that summarizes the distribution's "uncertainty" or "information content."

Examples:
- Uniform on 256 symbols: \\(H = \log_2 256 = 8\\) bits per symbol.
- Two symbols, each probability 1/2: \\(H = 1\\) bit. (Tell them which one — minimum cost.)
- Two symbols with probabilities 0.99 and 0.01: \\(H \approx 0.081\\) bits per symbol. Almost no uncertainty; you can compress.
- One symbol with probability 1: \\(H = 0\\). No uncertainty; nothing to send.

## Shannon's source coding theorem

**Theorem.** For any source \\(p\\) and any \\(\epsilon > 0\\):

1. **Achievability**: there exists a code with expected length per symbol less than \\(H(p) + \epsilon\\).
2. **Converse**: every uniquely decodable code has expected length per symbol at least \\(H(p)\\).

So \\(H(p)\\) is the minimum expected number of bits per symbol, and it is achievable in the limit of long blocks.

## The achievability proof: typical sequences

Block the input into chunks of \\(n\\) symbols. There are \\(|\mathcal{X}|^n\\) possible blocks, but most of them have probability essentially zero, and a small subset accounts for nearly all of the probability.

A block \\(x_1, x_2, \dots, x_n\\) is *typical* if

\\[ 2^{-n(H(p) + \epsilon)} \leq P(x_1, \dots, x_n) \leq 2^{-n(H(p) - \epsilon)} \\]

By the law of large numbers, the empirical entropy \\(-\frac{1}{n} \log P(x_1, \dots, x_n)\\) converges to \\(H(p)\\) almost surely as \\(n \to \infty\\). So with probability approaching 1, a sample is typical.

How many typical sequences are there? Each one has probability at most \\(2^{-n(H(p) - \epsilon)}\\), and they have total probability at most 1, so there are at most \\(2^{n(H(p) + \epsilon)}\\) typical sequences. (And at least \\(2^{n(H(p) - \epsilon)} \cdot (1 - \delta)\\) of them, by the matching probability bound.)

The compression: enumerate typical sequences and assign each a binary string of length \\(\lceil n(H(p) + \epsilon) \rceil\\). Encode atypical sequences with a flag bit followed by an uncompressed representation. By the law of large numbers, atypical sequences are rare and contribute negligibly to the average length. The expected per-symbol length is \\(H(p) + \epsilon + o(1)\\). \\(\blacksquare\\)

The proof is constructive in principle but exponentially expensive. Practical compressors use Huffman coding, arithmetic coding, or range coding to achieve the same bound efficiently.

## Huffman coding

Given a finite alphabet with known probabilities, Huffman's algorithm builds an optimal *prefix code* in \\(O(n \log n)\\) time:

1. Make each symbol a leaf with its probability.
2. Repeatedly merge the two lowest-probability nodes into a new node with their sum.
3. The resulting tree's root-to-leaf paths are the codewords (label left edges 0, right edges 1).

The expected codeword length is at most \\(H(p) + 1\\), within 1 bit of the entropy bound. By blocking symbols (encoding pairs or triples instead of individuals), the per-symbol overhead is amortized away.

Huffman coding is the standard for static-distribution compression. JPEG uses it. ZIP's DEFLATE uses Huffman after LZ77.

## Arithmetic coding

A more sophisticated approach: represent the entire message as a single number in [0, 1), narrowing the range based on probabilities of each successive symbol.

For each symbol, the current interval \\([\ell, r)\\) is split proportionally to the probabilities, and \\([\ell, r)\\) is updated to the sub-interval corresponding to the observed symbol. After encoding, output any binary fraction in the final interval. The number of bits needed is approximately \\(-\log_2(\text{final interval length}) = -\log_2 \prod_i p(x_i) = -\sum_i \log_2 p(x_i)\\), the empirical information content.

For a long message, this is exactly \\(n H(p) + O(1)\\), within a constant of the optimum *regardless of the alphabet size*. Arithmetic coding does not have Huffman's "1 bit overhead" because it does not constrain itself to integer-bit-length codewords.

Arithmetic coding (and its modern successor, Asymmetric Numeral Systems) is used in advanced compressors: H.264 and H.265 video, HEVC's CABAC, BPG image format, Brotli, zstd's later modes.

## Universal compression

The source coding theorem assumes you know \\(p\\). For real data — text, source code, structured documents — you do not.

*Universal* compressors achieve the entropy bound asymptotically without knowing the source distribution. The Lempel-Ziv family (LZ77, LZ78, LZW) maintains a dictionary of seen substrings and represents each new substring by a reference to the dictionary. As the message grows, the dictionary captures the source's structure, and the compression rate approaches the entropy.

This is the basis of gzip, brotli, and most general-purpose compression. The lossless output is, asymptotically, within a constant factor of the source's true entropy, even though gzip has no idea what English (or your specific data) actually is.

## Why this is wonder

Before Shannon, "how compressible is data?" was a folk question. People knew some data was more compressible than other data, but no one had isolated the right quantity. Shannon's contribution was to define entropy and prove that this number — and only this number — was the right answer.

The proof has the structure of a duality. *Achievability* shows the bound is reachable. *Converse* shows you cannot go below it. Together they pin down the entropy as the exact compression limit. Most engineering problems do not have such tight characterizations; you compute upper and lower bounds and hope they match within a constant. Information theory has them matching exactly.

The same structure recurs throughout the field: channel capacity is the precisely right number for noisy-channel coding; rate-distortion bounds are precisely the right tradeoffs for lossy compression. Each time, the achievability proof uses random codes with typical-sequence arguments, and the converse uses Fano's inequality. The framework is as solid as a theory of physics: predictions match measurements, and the predictions are ahead of the engineering.

## The wonder, in concrete terms

A compressor that knows nothing about your data — gzip, with default parameters — squeezes English text down to about 30% of its original size. This is within a few percent of the entropy of English under reasonable models. The compressor cannot "understand" English; it just exploits that the same byte sequences recur. Yet the gap to optimal is small because LZ-based compressors track *all* recurrent structure, and entropy is precisely what the recurrence statistics measure.

When you read in a textbook that the entropy of English is "about 1.5 bits per character" you should pause. That number is the answer to a deep mathematical question about the language, computed from frequency tables. And it agrees, to within a small constant, with what general-purpose compressors achieve in practice. The information-theoretic bound and the engineering achievement match. This is rare and it is wonder.

## Where to go deeper

- Shannon, *A Mathematical Theory of Communication*, Bell System Technical Journal, 1948. The original. Read sections 1 through 9.
- Cover and Thomas, *Elements of Information Theory* (2nd ed.). The textbook. Chapter 5 is the source coding theorem.
