# Compressed sensing

If a signal is sparse — most of its coefficients in some basis are zero — you can recover it from many fewer measurements than the Nyquist rate would suggest. Specifically: a signal of length \\(n\\) with only \\(k\\) non-zero coefficients can be reconstructed exactly from about \\(O(k \log(n/k))\\) random linear measurements, even though the system of equations is wildly underdetermined. The reconstruction is the solution of a convex optimization problem and it is provably correct.

This contradicts the intuition every engineer absorbs in a signals course. The Nyquist rate, the sampling theorem, the idea that you need at least one measurement per degree of freedom — all of those are about *general* signals. Most real signals are not general. They are sparse, and the math knows it.

## The setup

A signal \\(x \in \mathbb{R}^n\\) is *k-sparse* if at most \\(k\\) of its entries are non-zero. (More generally, sparse in some basis: \\(x = \Psi s\\) where \\(\Psi\\) is an orthonormal basis and \\(s\\) is sparse. JPEGs are sparse in the DCT basis. Natural images are approximately sparse in wavelet bases.)

Take \\(m\\) linear measurements:

\\[ y = A x \\]

where \\(A\\) is an \\(m \times n\\) matrix and \\(y \in \mathbb{R}^m\\). With \\(m \ll n\\), this is a hugely underdetermined system. Infinitely many \\(x\\) satisfy any given \\(y\\). Classical linear algebra: you cannot recover \\(x\\) from \\(y\\). End of story.

Compressed sensing says: if \\(x\\) is sparse and \\(A\\) is suitably random, you *can* recover it exactly, by solving

\\[ \min_x \|x\|_1 \quad \text{subject to} \quad Ax = y \\]

Convex optimization. Polynomial time. Exact recovery of \\(x\\), with overwhelming probability.

## Why \\(\ell_1\\), not \\(\ell_0\\)

The problem you would naively pose is

\\[ \min_x \|x\|_0 \quad \text{subject to} \quad Ax = y \\]

where \\(\|x\|_0\\) counts non-zero entries. This finds the sparsest signal consistent with the measurements. It is also NP-hard. You would have to try all \\(\binom{n}{k}\\) possible sparsity patterns.

The breakthrough was that, under the right conditions on \\(A\\), the \\(\ell_1\\) minimization gives the same answer as \\(\ell_0\\) minimization. The \\(\ell_1\\) version is convex and solvable by linear programming or proximal-gradient methods.

The geometric intuition: \\(\ell_1\\) balls have corners at the coordinate axes. Sliding an \\(\ell_1\\) ball outward until it first touches the affine subspace \\(\{x : Ax = y\}\\), the contact point tends to be at a corner — i.e., a sparse vector. \\(\ell_2\\) balls are smooth and contact the subspace generically, giving non-sparse solutions. The pictorial argument is the entire intuition.

```
   ell_1 ball: diamond, vertices on axes
   ell_2 ball: round
   The constraint set Ax=y is an affine subspace
   sliding outward.
   It first touches the diamond at a vertex (sparse).
   It first touches the circle on its surface (not sparse).
```

## The Restricted Isometry Property

Why does it work for some matrices and not others? Candès and Tao formulated the *Restricted Isometry Property*: \\(A\\) satisfies RIP of order \\(k\\) with constant \\(\delta_k\\) if for every \\(k\\)-sparse vector \\(x\\),

\\[ (1 - \delta_k) \|x\|_2^2 \leq \|Ax\|_2^2 \leq (1 + \delta_k) \|x\|_2^2 \\]

That is: \\(A\\) approximately preserves the lengths of all \\(k\\)-sparse vectors. It acts almost like an isometry on the union of all \\(k\\)-dimensional coordinate subspaces.

If \\(A\\) satisfies RIP of order \\(2k\\) with \\(\delta_{2k} < \sqrt{2} - 1\\), then \\(\ell_1\\) minimization recovers any \\(k\\)-sparse \\(x\\) exactly from \\(y = Ax\\). Approximate sparsity gives approximate recovery.

The miracle is that random matrices satisfy RIP. If \\(A\\) is filled with i.i.d. Gaussian entries scaled by \\(1/\sqrt{m}\\) (or other "incoherent" distributions), then for \\(m = O(k \log(n/k))\\), RIP of order \\(2k\\) holds with high probability.

## What measurements look like

The measurement matrix is dense and random. Each row is a random linear combination of all \\(n\\) entries of \\(x\\). Physically: each measurement is the inner product of \\(x\\) with a random pattern.

Concrete example, the *single-pixel camera*: a digital micromirror device (DMD) is a 2D array of mirrors that can each be flipped on or off. To "measure" the inner product of an image \\(x\\) with a random binary pattern, you set the DMD to the pattern, focus the entire reflected light onto a single photodiode, read the total intensity. That gives you one entry of \\(y = Ax\\). Repeat for each row of \\(A\\). After \\(m\\) measurements, solve the \\(\ell_1\\) problem to recover the image. The camera has *one* photodiode and produces megapixel images.

The MRI version: an MRI machine measures Fourier coefficients of a 2D slice of the body. Naïve sampling fills a grid of Fourier space; sampling time is proportional to the number of points. Compressed-sensing MRI samples a *random subset* of Fourier coefficients, then reconstructs the image by sparsity in the wavelet basis. Same image quality, fraction of the time. Adopted clinically; it is in production scanners now.

## Recovery in practice

Solving the \\(\ell_1\\) minimization is well-studied. ISTA, FISTA, primal-dual splitting, ADMM, SPGL1 — many algorithms with provable convergence and good empirical performance. For \\(n\\) in the millions and \\(m\\) in the hundreds of thousands, current solvers run in seconds on a laptop.

In practice, signals are not exactly sparse but approximately sparse: most coefficients are small but not zero, and the signal is dominated by a few large ones. Compressed sensing handles this gracefully: recovery error is bounded by the tail of the sorted coefficient magnitudes, with no penalty if the signal is exactly sparse.

Noise is also handled: if \\(y = Ax + e\\) with \\(\|e\|_2 \leq \epsilon\\), the relaxed problem

\\[ \min_x \|x\|_1 \quad \text{subject to} \quad \|Ax - y\|_2 \leq \epsilon \\]

gives recovery error proportional to \\(\epsilon\\) and to the tail of the coefficient magnitudes.

## What this overturns

Engineering tradition before 2006: to digitize a signal of bandwidth \\(B\\), sample at rate \\(2B\\); to measure a signal in dimension \\(n\\), make at least \\(n\\) measurements. These rules are correct for general signals.

Compressed sensing: most signals you actually want to measure (images, audio, MR scans, sensor readings) are sparse in some basis. The rules above were leaving information on the table. You only need a number of measurements proportional to the signal's complexity (sparsity), not its ambient dimension.

This had immediate impact on hardware design. MR imaging, radio astronomy, hyperspectral imaging, single-pixel cameras, sub-Nyquist analog-to-digital converters — all gained measurement-cost reductions of an order of magnitude or more, justified by the math.

## The wonder

A signal of length one million with a hundred non-zero entries is a hundred-dimensional object hiding in a million-dimensional ambient space. The intuition is that to find it you would need a million measurements, because that is the size of the ambient space. The truth is that you need a few thousand carefully-randomized measurements, because you are looking for a *low-dimensional* object, and a million measurements would be wasteful. The mathematics tells you the right number, gives you the algorithm to recover, and proves it works with overwhelming probability — provided your measurement matrix has the right random structure.

The intuition that "you need at least one measurement per degree of freedom" was an artifact of thinking about signals as black boxes. Once you incorporate the structural assumption of sparsity, you escape it.

## Where to go deeper

- Candès and Wakin, *An Introduction to Compressive Sampling*, IEEE Signal Processing Magazine, 2008. The clear introductory survey.
- Foucart and Rauhut, *A Mathematical Introduction to Compressive Sensing*. The textbook treatment with the RIP and recovery proofs.
