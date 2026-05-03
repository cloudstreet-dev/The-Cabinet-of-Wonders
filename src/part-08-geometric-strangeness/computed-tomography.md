# Computed tomography

A CT scanner does not see inside your body. It cannot. X-rays go through you in straight lines, attenuated according to the densities they pass through, and the detectors on the far side measure only the *total* attenuation along each ray. From a sequence of these one-dimensional projection scans, taken at many angles around your body, the computer reconstructs a full three-dimensional image of your insides — every organ, every bone, every tumor, with millimeter resolution.

The mathematics is a 1917 result of the Austrian mathematician Johann Radon: a function on the plane is fully determined by its line integrals. The inversion is explicit. Sixty years later this turned into Cormack and Hounsfield's CT scanner (Nobel 1979), and forty years after that, every emergency room has one.

## The Radon transform

Let \\(f(x, y)\\) be a function on the plane (the unknown density of a 2D slice through the patient's body). The *Radon transform* of \\(f\\) is the function

\\[ R f(\theta, s) = \int_{L_{\theta, s}} f(x, y) \, d\ell \\]

where \\(L_{\theta, s}\\) is the line at angle \\(\theta\\) (from the \\(x\\)-axis) and signed distance \\(s\\) from the origin. So \\(R f(\theta, s)\\) is the integral of \\(f\\) along that line.

A CT scanner physically computes the Radon transform: the X-ray beam attenuates along a line; the detector reading (logarithm of attenuation) is the line integral of the density. The scanner samples \\(R f(\theta, s)\\) for many \\(\theta\\) (rotating gantry angle) and many \\(s\\) (offset from center).

## Radon's inversion theorem

Radon proved: \\(f\\) can be recovered from \\(R f\\), and provided an explicit formula. The modern statement uses the *filtered backprojection*:

\\[ f(x, y) = \frac{1}{2\pi} \int_0^\pi \left[ \int_{-\infty}^\infty \widehat{R f}(\theta, \omega) \, |\omega| \, e^{2 \pi i \omega s} \, d\omega \right]_{s = x \cos\theta + y \sin\theta} d\theta \\]

where \\(\widehat{R f}\\) is the 1D Fourier transform of \\(R f\\) in \\(s\\) at fixed \\(\theta\\).

In words: for each angle \\(\theta\\), Fourier-transform the projection in \\(s\\), apply a *ramp filter* (multiply by \\(|\omega|\\)), inverse-transform back, then "smear" the filtered projection back across the image (each line gets its filtered value spread along itself), and average over all angles.

The reason this works: the *Fourier slice theorem* says that the 1D Fourier transform of a projection at angle \\(\theta\\) equals the 2D Fourier transform of \\(f\\) restricted to the line through the origin at angle \\(\theta\\). So projections at many angles fill in the 2D Fourier transform of \\(f\\) along radial lines. Polar-to-Cartesian resampling, then inverse Fourier transform, recovers \\(f\\). The ramp filter is the Jacobian of the polar-to-Cartesian transformation.

## Filtered backprojection in practice

Real CT reconstruction does this with FFT-based fast inversion. For a slice with \\(N \times N\\) pixels and \\(M\\) projection angles, the cost is \\(O(M N \log N + M N^2)\\), or \\(O(N^3)\\) for \\(M = N\\). On modern hardware: a 512×512 slice reconstructs in milliseconds.

For 3D imaging, slices are reconstructed independently or with helical reconstruction algorithms (FDK, Katsevich) that handle non-planar X-ray paths. A full chest CT, ~1000 slices, reconstructs in seconds.

## What "iterative reconstruction" adds

Filtered backprojection is fast and good. Modern scanners do better with *iterative reconstruction*:

1. Start with a guess image \\(\hat{f}\\).
2. Forward-project (compute the model's Radon transform). Compare to measurements.
3. Adjust \\(\hat{f}\\) to reduce the difference, applying prior knowledge (smoothness, sparseness in some basis, edge preservation).
4. Iterate.

This handles low-dose data (less radiation), incomplete projections (limited-angle CT), and noise. The price is computational cost — minutes instead of milliseconds — but for reduced-radiation protocols, dose savings of 50% with comparable image quality are routine.

The mathematical scaffolding has features in common with compressed sensing: solve an inverse problem with side information about the structure of the solution.

## What CT actually measures

CT is a transmission measurement: density (linear attenuation coefficient) at the X-ray energy. Bones (high-Z elements like calcium) attenuate strongly. Soft tissue is mostly water; small differences distinguish liver from kidney. Air is nearly transparent. The reconstructed image is calibrated in *Hounsfield units*: water = 0, air = -1000, dense bone = +1000+. This linearization makes images comparable across scanners.

Multi-energy CT (dual-energy or photon-counting) acquires at multiple X-ray energies; from the relative attenuations, you can decompose tissues by atomic-number content (e.g., distinguish calcium from iodine contrast).

## Why this is a wonder

X-rays cannot see *what is inside* you. They go through. Each detector reading is one number — the total attenuation along the ray's path. From this stream of one-dimensional projections, the mathematics reconstructs a 3D image, with millimeter resolution, in seconds.

The reconstruction is *exact*, given enough projections. The Radon transform is invertible. Take enough angles, sample finely enough in \\(s\\), and the image is recovered exactly (in the noiseless infinite-sample limit). With finite samples and noise, you get an approximation whose quality is governed by sample density and signal-to-noise — and the modern engineering pushes both.

The wonder is in the asymmetry: from a series of "shadow" measurements (each a single number per ray), you reconstruct the full volumetric structure. The X-rays themselves never resolve to "this is a kidney"; the structure is purely a mathematical reconstruction.

## Other things this works for

The Radon transform applies to any setting where you measure line integrals of an unknown function:

- **MRI**: sampling Fourier coefficients (related to but not exactly Radon transform; uses a similar inverse-problem framework).
- **Seismic imaging**: integrating wave properties along propagation paths.
- **Astronomy (radio interferometry)**: each baseline measures one Fourier component of the sky brightness; reconstruction is similar in spirit.
- **Electron tomography**: cryo-EM uses Radon-transform-style reconstruction to build 3D protein structures from many 2D images.
- **Particle physics (track reconstruction)**: line-integral measurements through detectors.

Whenever your sensor only sees integrated quantities along straight-ish paths, the Radon transform (or a close cousin) is the inversion theorem.

## The wonder, in patient terms

You lie on a table; a giant donut spins around you for ten seconds; thirty seconds later a doctor sees a 3D map of your insides at sub-millimeter resolution. Inside that black box: X-rays attenuating along straight lines; detectors recording total attenuations; a computer applying Fourier transforms, ramp filters, and backprojection to thousands of 1D projections taken from hundreds of angles; a 1917 theorem of pure mathematics tying the whole thing together.

The theorem was published before the X-ray scanner that would exploit it had been imagined. Cormack and Hounsfield rediscovered the inversion problem in the 1960s, working independently of the original Radon paper. The mathematical foundation was sitting in the literature for decades, waiting for the engineering to catch up.

## Where to go deeper

- Radon, *Über die Bestimmung von Funktionen durch ihre Integralwerte längs gewisser Mannigfaltigkeiten*, Math.-Nat. Klasse, 1917. The original.
- Natterer, *The Mathematics of Computerized Tomography*, SIAM 2001. The reference textbook for the modern field.
