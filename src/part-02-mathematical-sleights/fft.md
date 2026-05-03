# The Fast Fourier Transform

Multiplying two polynomials of degree \\(n\\) by the obvious method takes \\(\Theta(n^2)\\) operations. The FFT does it in \\(\Theta(n \log n)\\) by transforming both polynomials into a representation where multiplication is pointwise — taking only \\(\Theta(n)\\) — then transforming back. The transform itself takes \\(\Theta(n \log n)\\), and you only have to pay it twice.

The astonishing part is that this is the algorithm Gilbert Strang called "the most important numerical algorithm of our lifetime," and it was hiding inside Gauss's notebooks since 1805 — published, but in Latin, in a posthumous miscellany, where no one looked for over a hundred and fifty years. Cooley and Tukey rediscovered it in 1965. Half of modern signal processing is a corollary.

## Why polynomial multiplication should be slow

A polynomial \\(a(x) = a_0 + a_1 x + \cdots + a_{n-1} x^{n-1}\\) is specified by its \\(n\\) coefficients. The product \\(c(x) = a(x) b(x)\\) has degree \\(2n - 2\\), and its coefficients are

\\[ c_k = \sum_{i + j = k} a_i b_j \\]

That is the convolution of the two coefficient sequences. Computed directly, it is \\(O(n^2)\\): \\(2n - 1\\) output coefficients, each a sum of up to \\(n\\) products.

There is no obvious way to do better. Each output coefficient depends on a different combination of inputs.

## The shift to a different representation

A polynomial of degree \\(< n\\) is determined by its values at any \\(n\\) distinct points (Lagrange interpolation). So you could specify \\(a\\) by \\((a(x_0), a(x_1), \dots, a(x_{n-1}))\\) for any choice of \\(n\\) distinct points. In this *value representation*, multiplication is trivial:

\\[ c(x_i) = a(x_i) \cdot b(x_i) \\]

\\(n\\) values multiplied with \\(n\\) values gives \\(n\\) values, in \\(O(n)\\) time. Add a few more sample points to handle the doubled degree, and the multiplication itself is fast.

The cost has shifted to *converting* between coefficient and value representations. Naively, evaluating a polynomial at \\(n\\) points takes \\(O(n^2)\\). Interpolating from values back to coefficients also takes \\(O(n^2)\\). We have not won anything.

Unless the evaluation points are chosen cleverly. The FFT picks the \\(n\\)th *roots of unity*: \\(\omega^0, \omega^1, \dots, \omega^{n-1}\\) where \\(\omega = e^{2\pi i / n}\\). With this choice, both transforms run in \\(O(n \log n)\\).

## The divide-and-conquer

Assume \\(n\\) is a power of 2. Split \\(a(x)\\) into even and odd parts:

\\[ a(x) = a_{\text{even}}(x^2) + x \cdot a_{\text{odd}}(x^2) \\]

where \\(a_{\text{even}}\\) and \\(a_{\text{odd}}\\) are polynomials of degree \\(< n/2\\) made from the even-indexed and odd-indexed coefficients of \\(a\\). To evaluate \\(a\\) at all \\(n\\) roots of unity, we need to evaluate \\(a_{\text{even}}(\omega^{2k})\\) and \\(a_{\text{odd}}(\omega^{2k})\\) for each \\(k\\).

Here is the magic: \\(\omega^{2k} = (\omega^2)^k\\), and \\(\omega^2\\) is a primitive \\((n/2)\\)-th root of unity. So evaluating \\(a_{\text{even}}\\) at the \\(n\\)-th squares is the same as evaluating \\(a_{\text{even}}\\) at the \\((n/2)\\)-th roots of unity. That is a smaller version of the same problem.

Furthermore, \\(\omega^{n/2} = -1\\), so for \\(k\\) and \\(k + n/2\\) we have \\(\omega^{k + n/2} = -\omega^k\\). The two evaluations \\(a(\omega^k)\\) and \\(a(\omega^{k + n/2})\\) share their subproblem evaluations and differ only in a sign:

\\[ a(\omega^k) = a_{\text{even}}((\omega^2)^k) + \omega^k \cdot a_{\text{odd}}((\omega^2)^k) \\]
\\[ a(\omega^{k + n/2}) = a_{\text{even}}((\omega^2)^k) - \omega^k \cdot a_{\text{odd}}((\omega^2)^k) \\]

So \\(n\\) evaluations of \\(a\\) reduce to \\(n/2\\) evaluations each of \\(a_{\text{even}}\\) and \\(a_{\text{odd}}\\), plus \\(n\\) constant-cost combine steps.

\\[ T(n) = 2 T(n/2) + O(n) \\]

\\[ T(n) = O(n \log n) \\]

The same recursion in reverse — and with \\(\omega^{-1}\\) substituted for \\(\omega\\) — gives the inverse transform, also \\(O(n \log n)\\).

## The butterfly diagram

A radix-2 FFT on 8 inputs:

```
a[0] ---+----+-----+-----> A[0]
        \  /\     /
a[4] ---+----+    \
        /  \/      \
a[2] ---+----+----+--+--> A[1]
        \  /\    /  /
a[6] ---+----+   \ /
                  X
a[1] ---+----+   / \
        \  /\   /   \
a[5] ---+----+-+      \
                       \
a[3] ---+----+----+     \
        \  /\    /       \
a[7] ---+----+    \       \
                          A[7]
```

(The actual diagram is denser, but this is the shape: each stage halves the problem size; each level has \\(n\\) butterflies; there are \\(\log_2 n\\) levels.)

## Why convolution becomes multiplication

The Discrete Fourier Transform of a sequence \\((a_0, \dots, a_{n-1})\\) is

\\[ A_k = \sum_{j=0}^{n-1} a_j \omega^{jk}, \quad \omega = e^{-2\pi i / n} \\]

This is exactly polynomial evaluation: \\(A_k = a(\omega^k)\\). The DFT *is* evaluating a polynomial at the \\(n\\)-th roots of unity.

The convolution theorem follows: if \\(c\\) is the convolution of \\(a\\) and \\(b\\), then the polynomials satisfy \\(c(x) = a(x) b(x)\\), so \\(c(\omega^k) = a(\omega^k) b(\omega^k)\\), so the DFT of the convolution is the pointwise product of the DFTs. To convolve two sequences, transform both, multiply pointwise, transform back.

This is why the FFT eats every domain that has a convolution in it. Audio filtering. Image filtering. Multiplication of large integers (Schönhage-Strassen, and faster variants). Polynomial multiplication. Solving differential equations on periodic domains. Cross-correlation in radar and astronomy. Any operation that has the form "a sliding-window weighted sum" is a convolution, and any convolution is fast on the FFT.

## Big-integer multiplication

Two \\(n\\)-digit integers can be multiplied in \\(O(n \log n)\\) bit operations. Treat each integer as a polynomial in its base; multiply by FFT; carry propagate (the only place that the integer constraint shows up). The grade-school \\(O(n^2)\\) algorithm we all learned is, asymptotically, beaten by the FFT-based one as soon as \\(n\\) is large enough — typically a few hundred digits. Modern arbitrary-precision arithmetic libraries switch to FFT-based multiplication for very large numbers.

Until 2019, the best known algorithm for integer multiplication was \\(O(n \log n \cdot \log \log n)\\) (Fürer). Harvey and van der Hoeven proved a version with the second log gone: \\(O(n \log n)\\) flat. This matches a 1971 conjecture by Schönhage and Strassen and is widely believed to be optimal.

## What changed when Cooley and Tukey published

Before 1965, computing a Fourier transform on \\(n\\) points was \\(O(n^2)\\). After 1965, it was \\(O(n \log n)\\). Spectral analysis of long signals went from minutes to milliseconds on the same hardware. This unlocked routine use of frequency-domain methods in radar, seismology, MRI imaging (which is essentially an inverse 2D FFT of measurements), and digital communications.

It was a genuine algorithmic phase change. The same hardware, same data, same problem — but a problem now tractable whose previous tractability boundary had been an order of magnitude tighter. Some classes of computation that were physically impossible became routine overnight.

## The wonder

A polynomial in coefficient form and a polynomial in value form are the same polynomial. The FFT is the observation that, with the right evaluation points, switching between those representations costs less than doing the work in either of them. The algorithm is recursive in a way that is essentially trivial to write down. And yet, before 1965, every major engineering domain that needed Fourier analysis spent vastly more compute than it had to, because no one had noticed that the transform had a divide-and-conquer structure waiting to be exploited.

Gauss had it. He wrote it down in his notebooks. He used it to compute orbits. It was published in his collected works in 1866. No one connected the dots until Cooley and Tukey, working on detecting Soviet nuclear tests, needed it to be fast.

## Where to go deeper

- Cooley and Tukey, *An Algorithm for the Machine Calculation of Complex Fourier Series*, Mathematics of Computation, 1965. The paper. Five pages.
- Heideman, Johnson, Burrus, *Gauss and the History of the Fast Fourier Transform*, IEEE ASSP Magazine, 1984. The story of Gauss's prior discovery.
- Brigham, *The Fast Fourier Transform and Its Applications*. Engineering reference for what to actually do with one.
