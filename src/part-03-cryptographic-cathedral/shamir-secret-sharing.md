# Shamir secret sharing

You have a secret. You want to split it into \\(n\\) pieces such that any \\(k\\) of them, combined, reconstruct the secret — but any \\(k - 1\\) of them, combined, reveal *nothing* about it. Not partial information. Not narrowed search space. Nothing, in the information-theoretic sense.

The construction is two pages of high-school algebra, published by Adi Shamir in 1979. It is one of the most useful primitives in cryptography and one of the easiest to prove correct.

## The construction

Pick a prime \\(p\\) larger than the secret \\(s\\). Choose \\(k - 1\\) random elements \\(a_1, \dots, a_{k-1}\\) uniformly from \\(\mathbb{Z}_p\\). Define the polynomial

\\[ f(x) = s + a_1 x + a_2 x^2 + \cdots + a_{k-1} x^{k-1} \mod p \\]

Note that \\(f(0) = s\\). The polynomial has degree at most \\(k - 1\\) and is otherwise random.

To create \\(n\\) shares, evaluate \\(f\\) at \\(n\\) distinct non-zero points: \\(s_i = (i, f(i))\\) for \\(i = 1, \dots, n\\). Hand share \\(s_i\\) to participant \\(i\\).

To reconstruct, gather any \\(k\\) shares. They are \\(k\\) points on a polynomial of degree at most \\(k - 1\\). Such a polynomial is uniquely determined by \\(k\\) points (Lagrange interpolation). Compute \\(f(0)\\) by interpolation, recover \\(s\\).

## Why \\(k - 1\\) shares reveal nothing

This is the elegant part. Any \\(k - 1\\) shares correspond to \\(k - 1\\) points. There are exactly \\(p\\) polynomials of degree at most \\(k - 1\\) passing through any \\(k - 1\\) given points (one for each possible value of \\(f(0)\\)). Equivalently: for any candidate value \\(s' \in \mathbb{Z}_p\\) of the secret, there is *exactly one* polynomial of degree \\(k - 1\\) consistent with the \\(k - 1\\) known shares and \\(f(0) = s'\\).

So the conditional distribution of the secret given any \\(k - 1\\) shares is uniform over \\(\mathbb{Z}_p\\). Information-theoretically, the shares carry no information about \\(s\\). This is not a computational-hardness assumption; it holds against an adversary with infinite computing power.

## Lagrange interpolation, the explicit formula

Given \\(k\\) shares \\((x_1, y_1), \dots, (x_k, y_k)\\), the secret is

\\[ s = f(0) = \sum_{i=1}^{k} y_i \prod_{j \neq i} \frac{-x_j}{x_i - x_j} \mod p \\]

The Lagrange basis polynomials evaluated at zero. Each share is multiplied by a fixed coefficient (depending only on which \\(x\\) values are present, not on the \\(y\\) values), and the secret is the weighted sum.

This means reconstruction is *linear* in the shares. Two implications:

- It composes cleanly: secret-sharing two values and adding the shares produces shares of the sum.
- The reconstruction coefficients can be precomputed once for any chosen subset of \\(k\\) participants.

## The picture

For \\(k = 2\\), the polynomial is a line. Two points fix the line. Knowing one point gives no information about \\(f(0)\\): the line could pass through any \\(y\\)-value at \\(x = 0\\). The threshold case \\(k = 2\\) is sometimes called "two-out-of-\\(n\\) sharing."

```
   secret s = f(0)
   ^
   |   (x_1, y_1)
   |    *
   |       *
   |          *  <- if you only know one point, line could
   |             *   pass through any y-value at x=0
   |                *
   +----------------------------> x
   0    1    2    3    4    5
```

For \\(k = 3\\), the polynomial is a parabola. Two points fix infinitely many parabolas; three points determine the parabola exactly. Same picture, one dimension up.

## Why polynomials, not just XOR?

Two-out-of-two sharing has a simpler form: pick random \\(r\\), share \\(r\\) and \\(s \oplus r\\). Either share alone is uniform; both together XOR to \\(s\\). This works only for \\(k = n = 2\\).

For threshold sharing with \\(k < n\\), you need something where any \\(k\\) determine the secret and any \\(k - 1\\) do not. Polynomials over a finite field have exactly this property (by the dimension argument: a polynomial of degree \\(k - 1\\) lies in a \\(k\\)-dimensional space, so \\(k\\) constraints fix it, \\(k - 1\\) leave one degree of freedom uniformly random). XOR has no such structure.

You could try other constructions — Chinese-remainder-theorem-based sharing (Asmuth-Bloom), code-based sharing (Reed-Solomon, which is essentially the same as Shamir) — but Shamir's is the cleanest, and the linearity of polynomial evaluation makes it the natural fit for downstream cryptographic protocols.

## Where it shows up

- **Threshold signatures and threshold decryption.** Shamir-share the private key. To sign or decrypt, gather \\(k\\) shareholders to compute the signature collectively without reconstructing the key. Used in threshold ECDSA, threshold BLS, distributed key management for cryptocurrency custody.
- **Secure multi-party computation.** BGW protocol uses Shamir sharing to compute on shared data; addition is local on shares, multiplication requires one round of interaction.
- **Disaster recovery.** Split the master key for a backup encryption among \\(n\\) custodians; any \\(k\\) can recover. The cliché is the bank vault opening only when three of five officers are present; the modern version is HSM and SSH key recovery.
- **Verifiable secret sharing (VSS).** Shamir alone trusts the dealer to give consistent shares. With added commitments (Feldman, Pedersen), participants can verify their shares are consistent without a trusted dealer. Foundation of modern threshold protocols.

## The information-theoretic claim, restated

This deserves emphasis because cryptography mostly trades in computational assumptions, and Shamir's scheme does not.

Shamir sharing is information-theoretically secure. There is no assumption about hardness of factoring, discrete log, lattice problems, or anything else. An adversary with infinite computing power, given \\(k - 1\\) shares, has no statistical advantage over an adversary with no shares at all. Both face a uniform distribution over the secret.

This makes Shamir sharing one of the few cryptographic primitives in real use that does not rest on any unproven assumption. The proof is direct counting, taking one paragraph.

## The wonder

You have a secret. You produce \\(n\\) random-looking numbers from it. Any \\(k\\) of them recompute the secret to its last bit; any \\(k - 1\\) of them tell an adversary literally nothing about the secret, even an adversary with unlimited computing power. The construction is six lines of polynomial arithmetic.

The depth of the wonder is in the dimension count. A polynomial of degree \\(k - 1\\) lives in a \\(k\\)-dimensional vector space. Each share is one linear constraint. The secret \\(f(0)\\) is one specific linear functional on that space. With fewer than \\(k\\) constraints, the value of any linear functional is uniformly distributed, by the obvious counting argument. So information-theoretic security follows from "linear algebra over finite fields preserves uniformity in unrestricted dimensions." The cryptography hides under the algebra; the algebra was waiting.

## Where to go deeper

- Adi Shamir, *How to Share a Secret*, Communications of the ACM, 1979. Two pages.
- Beimel, *Secret-Sharing Schemes: A Survey* (2011). Modern landscape, including verifiable and proactive variants.
