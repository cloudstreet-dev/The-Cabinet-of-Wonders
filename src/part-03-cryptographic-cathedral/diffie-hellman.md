# Diffie–Hellman key exchange

The 1976 paper that introduced public-key cryptography included, almost as a side remark, a key-agreement protocol. Two people, who have never communicated before, can shout one number each into a public channel, and from that exchange compute a shared secret. An eavesdropper records both numbers in full and cannot reconstruct the secret. The eavesdropper cannot even reduce the search space.

The whole protocol fits in three lines of arithmetic. It is the canvas on which most of the rest of the cathedral is painted.

(See also *Public-key cryptography*, which describes the broader picture this construction sits inside.)

## The protocol

Public parameters: a large prime \\(p\\) and a generator \\(g\\) of a large subgroup of \\(\mathbb{Z}_p^*\\).

- Alice picks a secret \\(a\\) uniformly at random and sends \\(A = g^a \bmod p\\).
- Bob picks a secret \\(b\\) uniformly at random and sends \\(B = g^b \bmod p\\).
- Both compute the shared secret \\(s = g^{ab} \bmod p\\). Alice computes it as \\(B^a\\); Bob computes it as \\(A^b\\); they agree because \\((g^b)^a = (g^a)^b = g^{ab}\\).

The eavesdropper Eve sees \\(p, g, A, B\\). To recover \\(s\\), she would have to solve the *Computational Diffie–Hellman problem*: given \\(g, g^a, g^b\\), compute \\(g^{ab}\\). The strongest publicly-known attack on CDH in a generic group of size \\(N\\) takes \\(O(\sqrt{N})\\) operations (Pollard's rho on the discrete log, then exponentiation). For \\(p\\) of 2048 bits with a properly chosen subgroup, this is far beyond classical computation.

That is the entire protocol. Two messages. Two exponentiations on each side.

## The hardness assumption

Diffie–Hellman security rests on two related assumptions:

**Discrete log (DL).** Given \\(g\\) and \\(g^x\\), find \\(x\\). Believed hard.

**Computational Diffie–Hellman (CDH).** Given \\(g, g^a, g^b\\), find \\(g^{ab}\\). Believed hard.

CDH is no harder than DL — if you can take logs you can solve CDH. Whether they are equivalent in general is unknown for arbitrary groups, although they are equivalent for many specific groups (Maurer-Wolf for groups with smooth-order auxiliary group structure).

For real-world security one wants something even stronger: the *Decisional Diffie–Hellman problem* (DDH). Given \\(g, g^a, g^b, h\\), can you tell whether \\(h = g^{ab}\\) or \\(h\\) is uniformly random in the group? In groups where DDH is hard, the shared secret \\(g^{ab}\\) is computationally indistinguishable from a uniform group element, so it can be hashed and used as a key with no leakage.

DDH is hard in some groups (subgroups of \\(\mathbb{Z}_p^*\\) of prime order; suitable elliptic curves) and *easy* in others (the full group \\(\mathbb{Z}_p^*\\) when \\(p \equiv 3 \mod 4\\), because of the Legendre-symbol leak). Choosing the right group is part of the engineering.

## Elliptic-curve Diffie–Hellman

Modern Diffie–Hellman runs on elliptic curves, not over \\(\mathbb{Z}_p^*\\). The reason: index-calculus attacks on the discrete log in \\(\mathbb{Z}_p^*\\) run in subexponential time \\(\exp(O((\log p)^{1/3} (\log \log p)^{2/3}))\\), so to be secure you need huge primes (3072 or more bits for 128-bit security). On well-chosen elliptic curves, no subexponential attack is known; only generic \\(O(\sqrt{N})\\) attacks apply, so a 256-bit curve gives 128-bit security.

The Curve25519 family is the dominant choice. The curve is

\\[ y^2 = x^3 + 486662 x^2 + x \quad \text{over } \mathbb{F}_{2^{255} - 19} \\]

with a designated base point \\(G\\). The DH operation is scalar multiplication: secret \\(a\\), share \\(aG\\); both parties compute \\(abG\\). The Montgomery ladder gives constant-time scalar multiplication that is safe against timing attacks. Curve25519 is chosen so the curve and its twist both have prime order, eliminating subgroup-confinement attacks; so it is implementable with simple, branch-free code. (Bernstein, 2006.)

## Authenticated key exchange

Plain Diffie–Hellman has a fatal weakness against an active adversary: man-in-the-middle. Eve intercepts \\(A\\) from Alice and replaces it with her own \\(A' = g^{a'}\\); intercepts \\(B\\) from Bob and replaces it with \\(B' = g^{b'}\\). Alice and Eve agree on \\(g^{ab'}\\); Bob and Eve agree on \\(g^{a'b}\\). Alice and Bob each *think* they are talking to the other but are actually talking to Eve, who relays messages, decrypting and re-encrypting. The eavesdrop is total.

Real protocols layer authentication on top. In TLS 1.3, the server signs the handshake transcript with its long-term private key; the client verifies the signature against a certificate chain rooted in a trusted CA. The signature ensures the \\(g^b\\) the client sees actually came from the server, not from a man-in-the-middle. The DH itself provides forward secrecy: even if the long-term key leaks later, past sessions stay secure because the ephemeral \\(a, b\\) are forgotten.

In Signal, the long-term identity keys plus a per-session ephemeral exchange give the *X3DH* protocol, which provides authenticated key agreement plus forward secrecy plus deniability (the long-term keys never sign anything; everything is derived through DH). The construction is several DH operations stitched together with a key-derivation function, but each component is a Diffie–Hellman.

## Why DH and not RSA key transport?

There is a competing way to bootstrap a session key: the client picks a random key \\(k\\), encrypts it under the server's RSA public key, sends it. The server decrypts. Both parties have \\(k\\).

This works, but has a crippling weakness: if the server's RSA private key ever leaks, *every* recorded session is decryptable forever, because the recorded ciphertext contains \\(k\\) under the server's public key. There is no forward secrecy.

Diffie–Hellman with ephemeral keys (DHE or ECDHE) avoids this. The server's long-term key signs the handshake but does not encrypt the session key; the session key is derived from ephemeral DH values that are forgotten after the connection. Recording today's traffic and stealing the long-term key tomorrow does not give you the session keys.

This is why TLS 1.3 removed RSA key transport entirely. Every TLS 1.3 connection uses (EC)DH for the actual key agreement. RSA, where it appears, is only for signatures.

## The wonder

The protocol is a few lines. The math is high-school exponent rules, applied modulo a big prime. The security is grounded in a problem (discrete log) that mathematicians had been thinking about for centuries with no inkling it would matter for telecommunications. The result is that two strangers with no prior contact can derive a shared secret over a fully public channel, and the rest of cryptographic engineering is built on this foundation.

It is the simplest non-trivial fact in the whole cathedral, and the one without which nothing else could stand up.

## Where to go deeper

- Diffie and Hellman, *New Directions in Cryptography*, IEEE Transactions on Information Theory, 1976. Eight pages, the original paper.
- Bernstein, *Curve25519: new Diffie–Hellman speed records*, PKC 2006. The design choices behind the modern curve.
