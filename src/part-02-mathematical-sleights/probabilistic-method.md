# The probabilistic method

You can prove that a mathematical object with a particular property exists, without ever constructing one, by showing that a random object has the property with positive probability. The argument never names a single example. It does not need to. If a random one works with positive probability, at least one works.

Erdős used this in 1947 to settle Ramsey number bounds that had been open since the 1930s, and the method has been applied to so many existence questions in combinatorics, geometry, and number theory that it is now its own subject. The wonder is that "I can prove an X exists" and "I can hand you an X" are different theorems, and combinatorics is full of cases where the first is easy and the second is desperately hard.

## The structure

The pattern is simple:

1. Define a probability distribution over the candidate objects.
2. Show that, with positive probability under this distribution, a random sample has the desired property.
3. Conclude: at least one sample with the property must exist.

Step 2 is usually done by proving \\(P(\text{bad}) < 1\\), so that the complement has positive probability. The "bad" event is the union of many specific bad outcomes; bound it by union bound, by linearity of expectation, by second-moment methods, by the Lovász local lemma.

## Erdős on Ramsey numbers

The Ramsey number \\(R(k, k)\\) is the smallest \\(n\\) such that every 2-coloring of the edges of \\(K_n\\) contains a monochromatic clique of size \\(k\\). The upper bound \\(R(k, k) \leq \binom{2k-2}{k-1}\\) had been known. Erdős proved a lower bound:

\\[ R(k, k) > 2^{k/2} \\]

without exhibiting a single 2-coloring of any \\(K_n\\) avoiding monochromatic \\(k\\)-cliques.

The argument: take \\(n < 2^{k/2}\\). Color each edge of \\(K_n\\) red or blue independently with probability 1/2. Let \\(X\\) be the number of monochromatic \\(K_k\\)'s. By linearity,

\\[ E[X] = \binom{n}{k} \cdot 2 \cdot 2^{-\binom{k}{2}} = \binom{n}{k} 2^{1 - \binom{k}{2}} \\]

For \\(n < 2^{k/2}\\) and \\(k \geq 3\\), this is less than 1 (after a routine calculation). Since \\(X\\) is a non-negative integer, \\(E[X] < 1\\) implies \\(P(X = 0) > 0\\). So there exists a coloring with no monochromatic \\(K_k\\). So \\(R(k, k) > n\\).

The bound has been improved since, but only by polynomial factors. The exponential gap between \\(2^{k/2}\\) and \\(4^k\\) (the upper bound) has been open for 75 years. The probabilistic argument that gave the lower bound is still essentially the best.

## Tournaments with no king

A tournament is an orientation of the complete graph; for every pair, one beats the other. A tournament has *property* \\(S_k\\) if for every set of \\(k\\) players, some other player beats all of them. Such tournaments exist for every \\(k\\). The probabilistic-method proof: take a random tournament on \\(n\\) players (each edge oriented independently with probability 1/2). The probability a fixed set of \\(k\\) players is *not* dominated by some other player is

\\[ \left(1 - 2^{-k}\right)^{n - k} \\]

By union bound,

\\[ P(\text{property } S_k \text{ fails}) \leq \binom{n}{k} \left(1 - 2^{-k}\right)^{n - k} \\]

For \\(n\\) large enough, this is less than 1. Existence proven; tournament not exhibited.

## Why nonconstructive matters

The probabilistic method gives existence without algorithm. For some problems, that is the only thing known. For the Ramsey lower bound, we still cannot exhibit a 2-coloring of \\(K_n\\) for \\(n\\) close to \\(2^{k/2}\\) avoiding monochromatic \\(K_k\\)'s; the best explicit constructions give bounds of the form \\(\Omega(c^k)\\) for some \\(c < 2\\), which is exponentially worse than the random argument.

This is the wonder, twice over. First: a property must hold, even though we cannot point to a witness. Second: the probabilistic guarantee is, for many problems, *better* than what any known explicit construction gives. Randomness "knows" something that we, with our current techniques, do not.

## Derandomization, when it works

Sometimes a probabilistic existence proof can be turned into a polynomial-time algorithm. The key technique is the *method of conditional expectations*: instead of sampling all decisions randomly, fix them one at a time in the direction that keeps \\(E[X | \text{decisions so far}]\\) below 1. The conditional expectation can be computed exactly in many cases, so the algorithm is deterministic.

For Erdős's argument: process edges one at a time; for each edge, choose the color that minimizes the conditional expected number of monochromatic cliques. The final coloring has \\(X \leq E[X] < 1\\), so \\(X = 0\\). Polynomial-time, deterministic, finds a coloring.

This works for many basic probabilistic-method arguments. For others, like the Lovász local lemma, derandomization was open for decades; Moser's algorithm (2009) finally gave a constructive version, with one of the cleverest probabilistic arguments in the recent literature.

## The local lemma

The Lovász local lemma is the deepest probabilistic-method tool. Suppose you have a collection of "bad" events \\(B_1, \dots, B_n\\), each with probability at most \\(p\\), and each \\(B_i\\) is independent of all but at most \\(d\\) of the others. If

\\[ e \cdot p \cdot (d + 1) \leq 1 \\]

then with positive probability *no* bad event occurs. This is much stronger than the union bound, which would need \\(np \leq 1\\). The local lemma exploits limited dependence; the union bound does not.

The lemma is proven by induction on the events; it shows that the conditional probability of \\(B_i\\) given any subset of the other events is bounded, which by induction gives a positive probability that all are avoided.

It has applications across combinatorics — graph colorings, satisfiability, codes, geometric structures. Its existence proofs were notoriously not constructive until Moser's 2009 algorithm, which constructs the desired object by a randomized resampling procedure that is itself analyzed by an entropy-compression argument: an algorithm whose existence is proved by a probabilistic-method-style argument about its own randomness budget. The technique is not just a technique; it is a fixed point of itself.

## The wonder

A mathematician proves that an object with property \\(P\\) exists by writing down a probability distribution and computing an expectation. The proof is a few lines. There may be no known explicit construction; the existence proof may be the only proof. Nothing in the proof refers to any specific object — it operates entirely on the distribution. And yet, the conclusion is logically equivalent to "such an object can in principle be exhibited."

The first time you see this, it feels like cheating. The second time, it feels like a tool. By the tenth time, you understand: the probabilistic method is not "almost a proof" or "a heuristic that suggests a construction." It is a proof. It just happens to be a proof that does not require the prover to know the answer.

## Where to go deeper

- Alon and Spencer, *The Probabilistic Method*. The reference. Read it cover to cover; every chapter introduces a new technique with worked examples.
- Moser and Tardos, *A Constructive Proof of the General Lovász Local Lemma*, JACM 2010. The derandomization story for the local lemma.
