# Reed–Solomon codes

A QR code with a quarter of its black-and-white squares scribbled out in marker still scans correctly. A CD with a thumb-sized scratch through the data layer still plays. A satellite link 12 light-minutes away from Earth gets every byte through despite cosmic rays flipping bits along the way. The same code is doing the work in all three cases. It treats the message as values of a polynomial, transmits enough redundant evaluations that any \\(t\\) errors can be located *and* corrected, and recovers the polynomial — and hence the message — from what arrives.

Reed and Solomon described the construction in 1960, in five pages. It has been the dominant industrial error-correcting code ever since.

## The setup

Pick a finite field \\(\mathbb{F}_q\\) (in practice usually \\(\text{GF}(256)\\), a field with 256 elements, where each element is a byte). Pick \\(n\\) distinct points \\(\alpha_1, \dots, \alpha_n \in \mathbb{F}_q\\), \\(n \leq q\\).

A Reed-Solomon code of length \\(n\\) and dimension \\(k\\) treats a message \\(m = (m_0, m_1, \dots, m_{k-1}) \in \mathbb{F}_q^k\\) as the coefficients of a polynomial:

\\[ p(x) = m_0 + m_1 x + m_2 x^2 + \cdots + m_{k-1} x^{k-1} \\]

The codeword is the vector of evaluations:

\\[ c = (p(\alpha_1), p(\alpha_2), \dots, p(\alpha_n)) \in \mathbb{F}_q^n \\]

The code has *rate* \\(k/n\\): each codeword carries \\(k\\) symbols of information in \\(n\\) symbols transmitted. The redundancy is \\(n - k\\).

## The error correction capacity

Two distinct polynomials of degree \\(< k\\) can agree on at most \\(k - 1\\) points (a non-zero polynomial of degree \\(< k\\) has at most \\(k - 1\\) roots, so the difference of two such polynomials has at most \\(k - 1\\) zeros). Hence any two distinct codewords differ in at least \\(n - k + 1\\) positions. The *minimum distance* \\(d\\) of the code is exactly \\(n - k + 1\\).

A code with minimum distance \\(d\\) can:
- *detect* up to \\(d - 1\\) errors (any error pattern of weight \\(< d\\) lands on a non-codeword, so the receiver knows something is wrong).
- *correct* up to \\(\lfloor (d-1)/2 \rfloor\\) errors (the received word is closer to one specific codeword than to any other).
- *correct* up to \\(d - 1\\) erasures (positions known to be missing; just interpolate the remaining \\(n - (d - 1) = k\\) good positions).

Reed-Solomon codes hit the *Singleton bound* \\(d \leq n - k + 1\\) with equality. They are *MDS codes* (Maximum Distance Separable). No code with the same \\(n, k\\) can correct more errors.

## Decoding

Receive a corrupted codeword \\(r \in \mathbb{F}_q^n\\). At least \\(n - t\\) positions agree with the original codeword \\(c\\), where \\(t\\) is the number of errors.

The decoding problem: find a polynomial of degree \\(< k\\) that agrees with \\(r\\) on at least \\(n - t\\) positions. This is the *polynomial reconstruction* problem, and several algorithms solve it.

**Berlekamp-Massey / Berlekamp-Welch (\\(t \leq (n-k)/2\\))**: classical algorithm. Find an *error locator polynomial* whose roots are the error positions; once located, use Lagrange interpolation on the remaining positions. Algorithmically straightforward; runs in \\(O(n^2)\\) field operations, faster with FFT-based variants.

**List decoding (Sudan, Guruswami-Sudan)**: relax to a list of candidates instead of a unique answer. List decoders correct up to \\(n - \sqrt{nk}\\) errors, more than the unique-decoding radius, by allowing the decoder to return a small list of candidates rather than a single one. Used in some applications where additional context can disambiguate.

For practical Reed-Solomon (e.g., \\((255, 223)\\) on bytes), decoding takes microseconds in software, even less in hardware.

## Why it is everywhere

Reed-Solomon was adopted everywhere it was needed because the trade-off matched everyone's needs:

- **CDs and DVDs**: the CIRC scheme (cross-interleaved Reed-Solomon coding) on a CD interleaves two RS codes with different parameters. Original CD redbook spec uses (32, 28) RS over GF(256). Burst errors (a scratch) span many adjacent symbols; interleaving spreads the burst across multiple codewords so each codeword sees only a few errors.
- **DVB and DAB (digital broadcasting)**: outer RS code outside an inner convolutional code; the convolutional code corrects most random errors, the RS cleans up the residual bursts.
- **QR codes**: RS over GF(256), with several error-correction levels (L, M, Q, H) trading message capacity for error tolerance. At level H, ~30% of the QR code can be lost or obscured.
- **Aztec, Data Matrix, MaxiCode, PDF417**: every 2D barcode standard uses Reed-Solomon.
- **Deep-space and satellite communications**: NASA's Voyager probes used a (255, 223) RS as outer code. Same family is used today on Mars rovers and Earth-orbit satellites.
- **RAID-6 and other erasure-coded storage**: RS over GF(256) for data redundancy. Two parity blocks per stripe (often called P and Q) are sufficient to recover from any two-disk failure.
- **Distributed-storage erasure codes**: Reed-Solomon and its variants in HDFS, Ceph, MinIO, AWS S3 all rely on RS-style polynomial encoding for cost-effective data redundancy.
- **Modern post-quantum cryptography candidates**: McEliece-type code-based cryptosystems use Reed-Solomon-like codes (or their generalizations like Goppa codes) as the secret structure.

## Erasure coding for storage

In the storage-as-erasure-coding setting, each disk holds one symbol of a Reed-Solomon codeword. \\(k\\) data disks plus \\(n - k\\) parity disks. Any \\(k\\) of the \\(n\\) disks suffice to reconstruct all the data — the receive list is just \\(k\\) known evaluations of a degree-\\(< k\\) polynomial, interpolate.

This gives extreme efficiency. Instead of triple-replication (3× storage), \\((10, 4)\\) RS gives 1.4× storage and tolerates 4 disk failures simultaneously. \\((20, 4)\\) gives 1.2× storage. Backblaze, AWS, and others have published their parameter choices.

The trade-off: encoding/decoding cost rises with \\(n - k\\), and during reconstruction the system reads from \\(k\\) other disks (potentially saturating the network). Trade-offs in encode/decode placement vs. cross-rack topology vs. recovery bandwidth motivate variants like *locally repairable codes* and *regenerating codes*.

## The math, in one paragraph

A polynomial of degree \\(< k\\) is determined by \\(k\\) of its values. So if you transmit \\(n > k\\) values, the receiver has \\(n - k\\) bits of redundancy — extra equations that the polynomial must satisfy. If errors corrupt fewer than \\(\lfloor (n-k)/2 \rfloor\\) values, the polynomial is still uniquely the closest one, and the locator-polynomial trick recovers it. If errors corrupt up to \\(n - k\\) erasure positions (positions known to be missing), the remaining \\(k\\) are exactly enough. The whole construction is just polynomial interpolation in a finite field, applied robustly. The rest is implementation detail and engineering.

## Where it has been replaced

For very long codes — gigabits to terabits — Reed-Solomon's \\(O(n^2)\\) decoding (or \\(O(n \log^2 n)\\) with FFT) becomes expensive. *LDPC codes* (Low-Density Parity-Check) and *turbo codes* approach the Shannon limit for noisy channels with linear-time iterative decoders. Modern wireless and DSL standards use LDPC instead of (or in addition to) Reed-Solomon.

But for short codes, for erasure correction, for predictable-failure-mode storage, Reed-Solomon remains dominant because:

- It is MDS — optimal in error-correction-per-redundancy.
- The math is exact, no probabilistic decoder failures.
- Implementation is simple enough for standard hardware.

## The wonder

Reed-Solomon turns a recovery problem into polynomial interpolation. The substrate is the finite field \\(\text{GF}(256)\\), which is just integer arithmetic mod a polynomial mod 2 — simple enough to fit in a microcontroller. The construction is published in five pages. It has been bulletproof in industrial use for 65 years and counting.

The wonder is that the same trick — represent your message as a polynomial, transmit redundant evaluations, recover by interpolation — works across every scale: from a millimeter-tall QR code to interplanetary radio signals. The wonder is also that the bound the codes achieve (Singleton) is tight, so there is no algorithmic improvement ahead; you can engineer faster decoders, but the math has settled the upper limit on what the codes can do.

## Where to go deeper

- Reed and Solomon, *Polynomial Codes Over Certain Finite Fields*, J. SIAM 1960. The original five-pager.
- Roth, *Introduction to Coding Theory* (Cambridge, 2006). Modern textbook with the full algorithmic picture.
