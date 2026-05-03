# Lossy compression

A two-megapixel photograph contains about six megabytes of raw pixel data. A JPEG of that photograph, indistinguishable to the eye, is about 200 kilobytes. Thirty times smaller, and you cannot tell. Audio compression squeezes a CD's 1.4 megabits per second down to 128 kilobits per second of MP3, again with most listeners unable to reliably tell the difference.

The compression is not finding redundancy in the data. It is throwing data away. Carefully. The trick is knowing what your eyes and ears do not actually look at.

## The setup

Lossless compression — gzip, FLAC — exploits redundancy. You can recover the original bit for bit. There are theoretical limits (Shannon's source coding theorem, with rate equal to the entropy of the source) and you cannot do better.

Lossy compression breaks past those limits by accepting reconstruction error. The question becomes: what is the smallest representation \\(\hat{x}\\) such that the perceived distortion \\(d(x, \hat{x})\\) is below threshold? That is rate–distortion theory, and Shannon also wrote the foundational paper for it (1959).

Different distortion measures yield different optimal codes. If you measure error by mean squared error in pixel space, you get one set of optimal codes — the wrong ones, because human vision does not measure error in pixel space.

## What human vision actually does

The retina has roughly 5 million color-sensitive cones (concentrated near the fovea) and 100 million luminance-sensitive rods (everywhere else). Color resolution is much lower than brightness resolution. The visual cortex is sensitive to spatial frequency and oriented edges; large smooth regions get coarse representation, while edges get fine representation. High spatial frequencies in chrominance (color) are essentially invisible.

JPEG is built around this list of what your eyes ignore.

## What JPEG does, step by step

**Change color space.** RGB is what monitors emit, but it is not how vision works. JPEG converts to YCbCr: one luminance channel Y and two chrominance channels Cb, Cr.

\\[ Y = 0.299 R + 0.587 G + 0.114 B \\]
\\[ C_b = (B - Y) / 1.772 + 0.5 \\]
\\[ C_r = (R - Y) / 1.402 + 0.5 \\]

**Subsample chrominance.** Because chrominance resolution is invisible at fine scales, JPEG averages 2×2 blocks of Cb and Cr down to a single value. This alone halves the data with no perceptible loss. (Notation: 4:2:0 chroma subsampling.)

**Cut into 8×8 blocks.** Each channel is divided into 8×8 pixel tiles. Each tile is compressed independently.

**Discrete cosine transform.** Each 8×8 tile is transformed by a 2D DCT into 64 frequency coefficients. The DCT is invertible; no information is lost yet. But the energy concentrates: most of the visual content is in a few low-frequency coefficients, and the high-frequency coefficients are usually small.

\\[ F(u, v) = \frac{1}{4} C(u) C(v) \sum_{x=0}^{7} \sum_{y=0}^{7} f(x,y) \cos\left[\frac{(2x+1)u\pi}{16}\right] \cos\left[\frac{(2y+1)v\pi}{16}\right] \\]

**Quantize.** This is where the loss happens. Divide each coefficient by a corresponding entry of a quantization matrix, round to integer.

```
Standard luminance quantization matrix (quality ~50):
 16  11  10  16  24  40  51  61
 12  12  14  19  26  58  60  55
 14  13  16  24  40  57  69  56
 14  17  22  29  51  87  80  62
 18  22  37  56  68 109 103  77
 24  35  55  64  81 104 113  92
 49  64  78  87 103 121 120 101
 72  92  95  98 112 100 103  99
```

The matrix is calibrated to perception. Low-frequency cells (top-left) have small divisors, preserving low-frequency content. High-frequency cells (bottom-right) have large divisors. After rounding, most high-frequency coefficients become zero. This is where the 30× compression comes from. The matrix is not arbitrary; it comes from psycho-visual experiments on what frequencies are below the visibility threshold for typical viewing distances.

The chrominance quantization matrix is even more aggressive — chroma high-frequencies are even less visible.

**Serialize.** The 8×8 quantized block is read in zig-zag order so that runs of zeros at the high frequencies become contiguous trailing zeros. Run-length encode them, then Huffman-code the result. This last stage is lossless and squeezes another factor of 2 to 4.

```
Zig-zag scan order:
  0  1  5  6 14 15 27 28
  2  4  7 13 16 26 29 42
  3  8 12 17 25 30 41 43
  9 11 18 24 31 40 44 53
 10 19 23 32 39 45 52 54
 20 22 33 38 46 51 55 60
 21 34 37 47 50 56 59 61
 35 36 48 49 57 58 62 63
```

To decode: undo each step. Multiply quantized coefficients by the quantization matrix (gaining only an approximation of the original DCT, since rounding lost precision), inverse DCT, upsample chroma, convert back to RGB. The errors that survive are the errors the human visual system was least sensitive to in the first place.

## What MP3 does

MP3 (MPEG-1 Audio Layer III) plays the same game in the audio domain. The two key facts about hearing:

1. **Frequency masking.** A loud tone at 1000 Hz raises the audibility threshold for nearby frequencies. A quiet 1100 Hz tone played simultaneously is inaudible.
2. **Temporal masking.** A loud sound raises the threshold for sounds slightly before and after it (forward masking lasts roughly 100 ms; backward, much shorter).

MP3 splits the audio into 32 frequency subbands, then further splits each subband by a Modified DCT into finer frequency bins. A psychoacoustic model computes a masking threshold for each bin in each frame, telling the encoder how much quantization noise can be hidden under audible signal. Bins that would be masked anyway get few bits or zero bits. Bins in the audible range get enough bits to keep quantization noise below the masking threshold.

The output is a bitstream where almost all the bits are spent on parts of the signal that you actually hear. The discarded parts include not just inaudible silences, but inaudible tones playing alongside loud ones, and pre-echo masked by transients. The compressed file genuinely contains less audio than the original, but the audio it contains is the audio you would have noticed anyway.

## What modern codecs add

JPEG and MP3 are 1990s codecs and they show their age. Modern lossy codecs:

- **HEIC, AVIF, WebP** (image): use intra-frame prediction (predict each block from neighbors and code only the residual), variable block sizes, more sophisticated transforms, and better entropy coders. Roughly 2× smaller than JPEG at the same quality.
- **Opus** (audio): adapts between two internal modes — a CELT-style transform coder for music, a SILK-style linear-prediction coder for speech — and chooses on the fly. Operates from 6 kbps speech to 510 kbps stereo with smooth transitions.
- **AV1, H.265, H.266** (video): generalize image-style block prediction to motion compensation across frames. The same psychovisual idea — quantize harder where you cannot see error — but applied to a four-dimensional signal (x, y, time, color).

The basic structure is the same in all of them. Transform to a domain where the signal is sparse. Quantize using a perceptually weighted matrix. Entropy-code the result.

## The wonder

A six-megabyte photograph and a two-hundred-kilobyte JPEG of the same scene look identical, but they are different files: byte for byte, the smaller one is missing 95% of the original data. The missing 95% was, by careful design, the parts your visual cortex was not going to attend to. The wonder is not the math — DCT, Huffman, entropy coding are all elementary. The wonder is that human vision and human hearing are predictable enough that you can build a quantization matrix that exactly matches the boundary of what fades from awareness, and on the other side of that boundary you can throw out anything you like.

## Where to go deeper

- Wallace, *The JPEG Still Picture Compression Standard*, IEEE Transactions on Consumer Electronics, 1992. Short, definitive, by the editor of the standard.
- Brandenburg, *MP3 and AAC Explained*, AES paper, 1999. From the engineer who led MP3.
