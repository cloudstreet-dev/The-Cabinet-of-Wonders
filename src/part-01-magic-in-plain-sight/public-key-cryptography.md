# Public-key cryptography

Two strangers, who have never met and share no prior secret, can talk over a wire that the entire world is listening to and end up with a secret that only they know. The eavesdropper hears every bit they exchange and learns nothing useful.

That is what happens every time your browser connects to a server. It happens in milliseconds, billions of times a day, and it depends on a piece of mathematics that did not exist before 1976.

## The setup that should be impossible

Until the mid-1970s every cipher in history had the same shape. Sender and receiver shared a key. The key had to be transported by some channel that the adversary could not read — a courier, a one-time pad mailed in advance, a face-to-face meeting. The encryption itself could be unbreakable; the problem was always that two people who wanted to talk privately had to first meet privately. For a global telephone system, or a planetary computer network, that is a non-starter.

Diffie and Hellman noticed something. The key-distribution problem looks symmetric, but if you are willing to use a one-way function — something easy to do and hard to undo — the symmetry breaks in your favor.

A one-way function \\(f\\) has the property that computing \\(f(x)\\) from \\(x\\) is fast, but computing \\(x\\) from \\(f(x)\\) is intractable. With one of those, the public-key world unfolds.

## The construction

Pick a public function \\(f\\) that is one-way. Each person generates a random secret \\(x\\) and publishes \\(f(x)\\). Now anyone in the world can encrypt a message to you using \\(f(x)\\), but only you, who knows \\(x\\), can decrypt it.

That is not a sleight. The receiver chooses their own secret. The sender uses public information to lock the message in a way that requires the receiver's secret to unlock. The eavesdropper, watching the wire, sees only public information and ciphertexts. Inverting \\(f\\) would let them in, but they cannot invert \\(f\\).

The cleanest concrete instance is RSA. Pick two large primes \\(p, q\\), multiply to get \\(n = pq\\). Choose a public exponent \\(e\\) coprime to \\(\varphi(n) = (p-1)(q-1)\\), and compute the private exponent \\(d \equiv e^{-1} \pmod{\varphi(n)}\\). The public key is \\((n, e)\\); the private key is \\(d\\).

Encryption of a message \\(m \in \mathbb{Z}_n\\):

\\[ c = m^e \bmod n \\]

Decryption:

\\[ m = c^d \bmod n \\]

Why does it work? By construction \\(ed \equiv 1 \pmod{\varphi(n)}\\), so \\(ed = 1 + k\varphi(n)\\) for some integer \\(k\\). Then by Euler's theorem,

\\[ c^d = m^{ed} = m^{1 + k\varphi(n)} = m \cdot (m^{\varphi(n)})^k \equiv m \cdot 1^k = m \pmod{n} \\]

Why is it secure? Recovering \\(d\\) from \\((n, e)\\) requires \\(\varphi(n)\\), which requires the factorization of \\(n\\). As far as anyone knows, factoring a 2048-bit RSA modulus is intractable on classical hardware. The construction trades the impossible problem of "share a secret over a public channel" for the merely-very-hard problem of "factor a big number."

## The deeper move

The most striking thing about RSA is not the algebra — it is the redefinition of what encryption can be. Pre-1976, encryption was a sealed envelope you handed to a trusted carrier. Post-1976, encryption is a mathematical lock whose closing mechanism you mail to the world. Anyone can close it; only you can open it.

That asymmetry is the whole game. It enables three things at once:

**Confidentiality.** Anyone encrypts to you with your public key.

**Authenticity.** You sign with your private key — compute \\(s = h(m)^d \bmod n\\) where \\(h\\) is a hash. Anyone can verify \\(s^e \equiv h(m) \pmod n\\). No one else could have produced \\(s\\) without knowing \\(d\\), so \\(s\\) is a signature only you could have made.

**Key agreement.** The Diffie–Hellman exchange itself: pick a public group \\(G\\) with generator \\(g\\). Alice picks secret \\(a\\) and sends \\(g^a\\). Bob picks secret \\(b\\) and sends \\(g^b\\). Both compute \\(g^{ab}\\). The eavesdropper sees \\(g^a\\) and \\(g^b\\) but cannot compute \\(g^{ab}\\) without solving the discrete log problem — find \\(a\\) from \\(g^a\\). They are stuck.

A version of the discrete-log story plays out today over elliptic curves rather than \\(\mathbb{Z}_p^*\\): same structure, smaller keys, faster math. The curve Curve25519 is the workhorse of TLS and SSH key exchange in the modern world.

## What the wire actually sees

When your browser hits an HTTPS site, the TCP handshake completes, then the TLS handshake begins. In the most common modern variant (TLS 1.3 with X25519):

```
Client                                              Server
------                                              ------
ClientHello { random, supported_curves,
              key_share = X25519(client_eph) } -->

       <-- ServerHello { random,
                         key_share = X25519(server_eph),
                         certificate (signed by CA chain),
                         signature over handshake transcript }

Both sides compute
  shared = X25519(client_eph_priv, server_eph_pub)
         = X25519(server_eph_priv, client_eph_pub)
HKDF(shared) derives symmetric session keys.
```

The certificate is the server proving its long-term identity, signed by a CA whose public key is baked into your operating system. The X25519 exchange is the ephemeral key agreement. The session keys derived from the agreement are used with a symmetric cipher (AES-GCM or ChaCha20-Poly1305) for the actual data — symmetric crypto is much faster, and the public-key step exists only to bootstrap a shared symmetric key over the open network.

Both halves are needed. Without certificates you have no idea who you agreed a secret with. Without ephemeral key agreement, recording today's traffic would let you decrypt it tomorrow if the server's long-term key ever leaks. Combining them gives forward secrecy: each session's key is derived from ephemeral randomness that is destroyed afterward, and an attacker who later compromises the server still cannot read past sessions.

## Why this is wonder, not just engineering

It is easy to lose the strangeness once you see TLS every day. Try this: explain to someone who does not know any cryptography how two people who have never met can shout in a crowded room and walk away with a secret only they know. Watch them try to figure out where the trick is. There is no trick. The hardness of factoring (or of discrete log) is doing the entire job. Mathematics that humans had been sharpening for two thousand years for its own sake turned out to contain the substrate of a worldwide private communication system, and we did not notice until the 1970s.

## Where to go deeper

- Diffie and Hellman, *New Directions in Cryptography* (1976). Eight pages. Read the original.
- Boneh and Shoup, *A Graduate Course in Applied Cryptography* (free online). Chapters on RSA, discrete log, and key exchange with the modern proof apparatus.
