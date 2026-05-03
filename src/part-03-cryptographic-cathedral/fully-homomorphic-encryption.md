# Fully homomorphic encryption

You can hand someone an encrypted version of your data, ask them to compute an arbitrary function on it, and receive back an encrypted answer that you alone can decrypt. The computer that ran the computation never saw any plaintext: not the inputs, not the intermediate values, not the output. They computed on ciphertexts directly, and what they got out was a ciphertext of the answer.

When Rivest, Adleman, and Dertouzos posed this in 1978 they called it "privacy homomorphisms" and openly suspected it might not exist. It existed for limited operations from the start (RSA preserves multiplication; ElGamal preserves multiplication too; Paillier preserves addition), but the *fully* homomorphic version — every computation, in any combination — was open until 2009. Craig Gentry's PhD thesis solved it.

The construction is bone-deep strange and the engineering is still maturing. It is the form of cryptography that, when it becomes fast enough, will let you outsource computation on your data to anyone in the world without having to trust them.

## What "homomorphic" means

An encryption scheme is *homomorphic* over an operation \\(\circ\\) if there is a corresponding ciphertext operation \\(\boxdot\\) such that

\\[ \text{Dec}(\text{Enc}(a) \boxdot \text{Enc}(b)) = a \circ b \\]

If you support both addition and multiplication on the underlying data, you can compute any boolean circuit (AND = multiplication, XOR = addition mod 2), so you can compute anything. The hard part is supporting both at once, in a way that does not blow up the ciphertext size.

## Partial homomorphisms: the easy cases

RSA is multiplicatively homomorphic: \\(\text{Enc}(a) \cdot \text{Enc}(b) = a^e b^e = (ab)^e \mod n = \text{Enc}(ab)\\). You can multiply ciphertexts without decrypting. But you cannot add them — RSA gives you only multiplication.

Paillier (1999) is additively homomorphic over \\(\mathbb{Z}_n\\): a particular product of ciphertexts decrypts to the sum of plaintexts. You can add but not multiply.

These were known and useful — Paillier is used in private-set-intersection protocols and electronic voting — but they each support only one operation. Compute a polynomial of degree \\(d\\) on Paillier-encrypted data, you cannot. Compute a sum of RSA-encrypted data, you cannot.

## Somewhat homomorphic

A scheme that supports both addition and multiplication, but only up to a limited number of operations, is *somewhat homomorphic*. The classical construction (Boneh, Goh, Nissim 2005) supports unlimited additions and one multiplication. Useful for some applications, fundamentally limited.

The reason for limits is *noise*. Modern lattice-based encryption hides the plaintext under random noise; decryption requires the noise to remain below a threshold. Each homomorphic operation amplifies the noise. After enough operations, decryption fails.

For example, in the BGV scheme over a learning-with-errors lattice problem:
- Initial noise on each ciphertext is small.
- Adding two ciphertexts adds the noises (linearly).
- Multiplying two ciphertexts multiplies them (quadratically in noise norm).

After a few multiplications, noise grows beyond the decryption threshold, and the ciphertext becomes garbage.

## Bootstrapping: the breakthrough

Gentry's 2009 thesis introduced *bootstrapping*. The idea: take a noisy ciphertext, encrypt it again under a fresh key (so it is *doubly* encrypted), then evaluate the *decryption circuit* of the inner key homomorphically. The result is a ciphertext under the outer key with the same plaintext but with fresh, low noise — independent of the original noise level.

Schematically:

```
Have: ciphertext c with high noise, decrypts under key sk1.
Goal: ciphertext c' with low noise, decrypts under key sk2.

Step 1: encrypt sk1 under sk2 to get an encrypted secret key, "the bootstrapping key."
Step 2: define decrypt(sk, c) as a circuit. Evaluate this circuit homomorphically
        on the encrypted sk1 and the (now public) c.
Step 3: the output is Enc_sk2(Dec_sk1(c)) = Enc_sk2(plaintext) = c'.
        Since decryption was evaluated on encrypted inputs, the noise of c'
        depends on the noise growth of the decryption circuit, not on the
        noise of c.
```

If the decryption circuit is shallow enough that the somewhat-homomorphic scheme can evaluate it without overflowing noise, bootstrapping works. The output ciphertext has fresh noise, and you can keep computing.

That is the fully homomorphic part. After every few operations, bootstrap to refresh noise. There is no hard limit on circuit depth.

The construction is delicate — the decryption circuit has to be simple enough, the parameters tight enough, the bootstrapping operation cheap enough — but it works, in principle and in practice. Gentry's original scheme bootstrapped in seconds; modern schemes (FHEW, TFHE) bootstrap in milliseconds or less.

## The lattice problems that hide the secrets

Modern FHE rests on the *Learning With Errors* problem (Regev 2005). LWE: given many noisy linear equations over \\(\mathbb{Z}_q\\) — \\((a_i, b_i = \langle a_i, s \rangle + e_i)\\) where \\(s\\) is secret and \\(e_i\\) is small noise — find \\(s\\). Easy without noise (Gaussian elimination). With noise from a discrete Gaussian distribution, conjectured to be hard, and proven hard under reductions to worst-case lattice problems.

LWE supports an additive homomorphism naturally: \\((a, b) + (a', b') = (a + a', b + b')\\) decrypts to \\(\langle a + a', s \rangle + (e + e') = \langle a, s \rangle + \langle a', s \rangle + (e + e')\\), so additions are linear in noise. Multiplication is more involved: ciphertexts become tensor products, and *relinearization* converts back to standard form. The post-multiplication noise is roughly the product of input noises plus a small fresh-key contribution.

Ring-LWE (Lyubashevsky-Peikert-Regev 2010) replaces vectors over \\(\mathbb{Z}_q\\) with elements of a polynomial ring \\(\mathbb{Z}_q[x]/(x^n + 1)\\), giving much better efficiency: ciphertexts encode \\(n\\) plaintexts at once via SIMD-like batching, and operations have nearly-linear cost in \\(n\\). Almost all practical FHE today is Ring-LWE-based.

## What you can actually do today

Real implementations exist:

- **TFHE / TFHE-rs**: gate-by-gate FHE; bootstraps after every Boolean gate. Slow per gate (~10 ms) but reliable for arbitrary circuits.
- **CKKS**: approximate-arithmetic FHE; treats ciphertexts as encoding fixed-point numbers and supports polynomial functions on them. Used for ML inference on encrypted data.
- **BGV / BFV**: integer-arithmetic FHE; SIMD-batched, good for vector operations like private database queries.

Performance: an encrypted matrix multiplication of moderate size is feasible in seconds. Encrypted neural-network inference (small models, e.g., ResNet) takes minutes. Encrypted training is far too slow at present. The asymptotic costs are now within an order of magnitude of cleartext for some workloads, and improving steadily.

Microsoft, IBM, Google, and various startups have FHE production deployments for narrow use cases: encrypted aggregations over health data, encrypted private information retrieval, encrypted cloud-stored database queries.

## Why this is the cathedral's keystone

FHE means *computation* and *trust* are independent. Today, to compute on data you give it to whoever you trust to run the computation. With FHE, you give the data to anyone, including untrusted parties — they can compute on it but cannot read it.

This dissolves the assumption that has sat under the design of every cloud system. Cloud providers have to be trusted because they hold your data in cleartext. With FHE, they would not. Your phone could send encrypted queries to a search engine that runs on its data without ever knowing what you searched. A medical service could compute on your encrypted DNA without seeing it. A bank could run risk models on encrypted positions across competitors who refuse to share the cleartexts.

The reason this is not yet how the cloud works is purely speed: FHE is currently 10× to 10000× slower than cleartext, depending on the operation. Closing that gap is an active engineering frontier. The mathematics is settled; the wonder is that it exists at all.

## The wonder

Before 2009, the question "can you compute arbitrary functions on encrypted data" was a 30-year-old open problem with no construction in sight, and respected cryptographers had publicly speculated it was impossible. The construction Gentry found in his thesis was inelegant — he himself called it "an impractical bootstrappable somewhat homomorphic scheme" — but it was a construction, and once you had one, the rest of the field could optimize. The scheme he described is now enormously faster than what he wrote, but the structure is the same. Encrypt with controlled noise. Run the decryption circuit homomorphically. Get fresh noise. Repeat.

It is one of the few cases in modern cryptography where a question that resisted decades of attention was answered with a construction that, although unwieldy, exhibited the desired property *for the first time*. After that, the engineering took over.

## Where to go deeper

- Craig Gentry, *Fully Homomorphic Encryption Using Ideal Lattices*, STOC 2009. The thesis-condensed paper. 8 pages.
- Halevi, *Homomorphic Encryption*, in *Tutorials on the Foundations of Cryptography* (2017). The cleanest pedagogical overview.
