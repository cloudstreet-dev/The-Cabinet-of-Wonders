# Slepian–Wolf coding

Two people each have a copy of a long document, but their copies have a small unknown set of differences — typos, a paragraph rewritten, scattered character changes. Person A wants to send their copy to Person B over a channel, using as few bits as possible. The intuitive approach: just send the differences. Easy.

But here is the harder version. Person A does *not know* what is on Person B's copy. They know only the *statistical relationship* between the two documents. Yet they can compress their message to only as many bits as the *conditional* entropy \\(H(A | B)\\) of their copy given B's — even though they cannot see B's copy.

That is the Slepian-Wolf theorem. It says distributed compression of correlated sources is no harder than centralized compression — just dramatically counterintuitive.

## The setup

Two random sources \\(X, Y\\) with joint distribution \\(p(x, y)\\). Encoder 1 has access to \\(X\\) only. Encoder 2 has access to \\(Y\\) only. Each encoder produces a binary message; the messages go to a single decoder that has both. The decoder reconstructs both \\(X\\) and \\(Y\\).

What is the smallest combined rate \\((R_X, R_Y)\\) achievable, in bits per source symbol?

If both encoders shared their data, the answer is the classical: \\(R_X + R_Y \geq H(X, Y)\\), the joint entropy. With sufficient block length, \\(H(X, Y)\\) is achievable.

If only one encoder has access to both sources (say encoder 1 has \\((X, Y)\\) and encoder 2 has \\(Y\\) alone, but they collaborate via a shared encoding), the answer is also \\(H(X, Y)\\): encoder 2 sends \\(H(Y)\\) bits encoding \\(Y\\); encoder 1, knowing both, conditionally encodes \\(X\\) given \\(Y\\) at rate \\(H(X | Y)\\). Total: \\(H(Y) + H(X | Y) = H(X, Y)\\).

The Slepian-Wolf result: the same total rate \\(H(X, Y)\\) is achievable *even when the encoders cannot communicate*. The achievable region is the polygon

\\[ R_X \geq H(X | Y), \quad R_Y \geq H(Y | X), \quad R_X + R_Y \geq H(X, Y) \\]

The corner point \\(R_X = H(X | Y), R_Y = H(Y)\\) is achievable: encoder 2 sends \\(Y\\) at rate \\(H(Y)\\) using a standard source code; encoder 1 compresses \\(X\\) to rate \\(H(X | Y)\\) without knowing \\(Y\\).

That last sentence is the wonder. *Compress \\(X\\) at rate \\(H(X | Y)\\) without knowing \\(Y\\).* The conditional entropy is the conditional entropy; the receiver has \\(Y\\), but the encoder does not.

## How is that possible

Encode \\(X\\) by hashing into bins of size \\(2^{nH(X|Y)}\\). Specifically, partition the typical \\(X\\)-sequences (of which there are \\(\approx 2^{nH(X)}\\)) into bins, with \\(\approx 2^{n(H(X) - H(X|Y))} = 2^{nI(X;Y)}\\) bins. Each \\(X\\)-typical sequence goes into one bin; encoding sends the bin index, which takes \\(n(H(X) - H(X|Y))\\) bits — wait, that is the wrong way.

Let me restart. Encoder 1 hashes \\(X\\) into one of \\(2^{nH(X|Y)}\\) bins. Each bin contains roughly \\(2^{nI(X;Y)}\\) typical \\(X\\)-sequences. So the encoding is \\(n H(X|Y)\\) bits long.

The decoder receives the bin index and \\(Y\\). It looks among the typical \\(X\\)-sequences in the indexed bin for the one most consistent with \\(Y\\) (i.e., jointly typical with \\(Y\\)). With high probability, exactly one such sequence exists in each bin (because \\(2^{nI(X;Y)}\\) candidates, but only \\(2^{n(I(X;Y))}\\) of all \\(X\\)-sequences are jointly typical with the observed \\(Y\\), so the expected count of jointly-typical candidates per bin is constant; with the right constants, decoding succeeds).

This is *random binning*. The proof structure is:

1. Encoder partitions typical sequences into random bins of the right size.
2. Encoder sends the bin index.
3. Decoder finds the unique bin element jointly typical with \\(Y\\).
4. By the union bound, the probability of more than one jointly typical candidate goes to zero as \\(n \to \infty\\).

The \\(X\\)-encoder needs to know the joint distribution \\(p(x, y)\\) — to define what "jointly typical" means and to design the binning — but does *not* need to know \\(Y\\). The asymmetry is not in what each side knows about the data; it is in what each side does with the bits.

## The constructive version

Random binning is non-constructive. To actually do Slepian-Wolf coding in practice, one of the standard tricks is to use *syndrome coding* with a linear error-correcting code.

Pick a linear code \\(C\\) with parity-check matrix \\(H\\), of dimension \\((n - k) \times n\\). Encoder 1 sends the *syndrome* \\(s = H X\\), an \\((n-k)\\)-bit vector. The decoder, knowing \\(Y\\) and the syndrome, treats \\(Y\\) as a noisy version of \\(X\\) and decodes the *coset* of \\(C\\) defined by syndrome \\(s\\) — the closest element to \\(Y\\) in that coset is the most likely \\(X\\).

For binary symmetric correlation between \\(X\\) and \\(Y\\) with crossover probability \\(p\\), and a code of rate \\(k/n\\) close to \\(1 - h_2(p) = 1 - H(X|Y)\\) (where \\(h_2\\) is the binary entropy), the decoding succeeds with high probability. Slepian-Wolf coding becomes channel coding for the conditional distribution \\(p(x | y)\\).

So you can do Slepian-Wolf with LDPC codes, turbo codes, polar codes — all the modern channel-coding apparatus.

## Wyner-Ziv: the lossy version

The lossy generalization (Wyner-Ziv 1976): if the decoder has \\(Y\\) as side information, and the encoder of \\(X\\) accepts distortion \\(D\\), what is the minimum rate? The answer is the *Wyner-Ziv rate-distortion function*, \\(R_{WZ}(D)\\), which equals the standard rate-distortion function \\(R_{X|Y}(D)\\) for many natural distortion measures and source distributions. Again: no penalty for lacking the side information at the encoder.

This is used in distributed video coding, where lightweight encoders (mobile cameras) compress with reference to side information available only at a powerful decoder.

## Where it shows up

- **Sensor networks**: many sensors record correlated data (the same field measured from different angles). Each sensor compresses against the global statistics without knowing the other sensors' data. Saves bandwidth substantially.
- **Distributed video**: low-power devices encode without doing motion estimation; the decoder uses already-decoded frames as side information.
- **DNA storage**: encoded data with known statistical structure is recovered with side information from reference sequences.
- **Heuristic application: rsync**: the basic rsync algorithm, where the receiver tells the sender hashes of blocks it already has, is not strictly Slepian-Wolf, but the underlying observation — distributed coding of correlated data — is the same.

## Why this is wonder

The intuition is that to compress \\(X\\) optimally given that the decoder has \\(Y\\), the encoder must know what \\(Y\\) is. Otherwise how does it know which redundancy to discard? Slepian-Wolf says: knowing the *statistics* is enough. The encoder does not need the actual \\(Y\\), only the joint distribution \\(p(x, y)\\). The compression matches what would be achievable if the encoder did know \\(Y\\).

The proof technique — random binning with jointly typical decoding — shows up everywhere in information theory once you have seen it once. It is the prototype for the *binning codes* that underlie multi-user information theory, including the Marton coding for broadcast channels, the side-information coding theorem, and several other distributed-source results.

The wonder, distilled: separation of *encoder knowledge* and *decoder knowledge* is not a barrier when the only thing the decoder needs is the relationship between the two. The encoder operates in *coset space* — sending residues modulo a sufficiently large coding lattice — and the decoder, with its side information, points uniquely to the correct coset element. The asymmetry in the protocol perfectly mirrors the asymmetry in available information.

## Where to go deeper

- Slepian and Wolf, *Noiseless Coding of Correlated Information Sources*, IEEE Transactions on Information Theory, 1973. The original.
- Cover and Thomas, *Elements of Information Theory*, Chapter 15.4. The clean modern proof.
