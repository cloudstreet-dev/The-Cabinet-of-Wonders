# Schwartz–Zippel

You have a polynomial in many variables, in some symbolic form so complicated that simplifying it to canonical form is computationally infeasible. You want to know if it is the zero polynomial. The trick: pick a random point, plug it in. If you get a non-zero value, the polynomial is not zero. If you get zero, the polynomial *probably* is zero, with a quantifiable failure probability you can drive arbitrarily low by repeating.

It is a randomized algorithm for an algebraic question, and it is the foundation under bipartite matching algorithms, polynomial identity testing, and a great deal of the SNARK ecosystem.

## The lemma

Let \\(P(x_1, \dots, x_n)\\) be a non-zero polynomial of total degree \\(d\\) over a field \\(F\\). Let \\(S \subseteq F\\) be any finite subset. If \\(r_1, \dots, r_n\\) are chosen independently and uniformly at random from \\(S\\), then

\\[ \Pr[P(r_1, \dots, r_n) = 0] \leq \frac{d}{|S|} \\]

So if you pick \\(|S| \geq 2d\\), a non-zero polynomial evaluated at a random point is zero with probability at most 1/2. With \\(|S| \geq 100d\\), probability at most 1/100. With independent repetitions, drive it as low as you like.

## Why this works

For a single-variable polynomial, the lemma is obvious. A degree-\\(d\\) polynomial in one variable has at most \\(d\\) roots. If you pick a random element of \\(S\\), at most \\(d\\) out of \\(|S|\\) choices land on a root.

For \\(n\\) variables, induct. Write \\(P\\) as a polynomial in \\(x_1\\) with coefficients that are polynomials in \\(x_2, \dots, x_n\\):

\\[ P(x_1, \dots, x_n) = \sum_{i=0}^{d_1} x_1^i \cdot Q_i(x_2, \dots, x_n) \\]

where \\(d_1\\) is the degree of \\(P\\) in \\(x_1\\). Some \\(Q_i\\) is nonzero; let \\(k\\) be the largest such index. The polynomial \\(Q_k\\) has total degree at most \\(d - k\\).

Two ways for \\(P\\) to vanish at a random point:

- \\(Q_k(r_2, \dots, r_n) = 0\\). By induction, this happens with probability at most \\((d - k)/|S|\\).
- \\(Q_k(r_2, \dots, r_n) \neq 0\\), but then \\(P\\) viewed as a single-variable polynomial in \\(x_1\\) is non-zero of degree \\(k\\), and \\(r_1\\) lands on one of its at most \\(k\\) roots, with probability at most \\(k/|S|\\).

Union bound: \\(\Pr[P = 0] \leq (d - k)/|S| + k/|S| = d/|S|\\). The induction closes.

## Polynomial identity testing

You want to know if two polynomials are equal: \\(P(x_1, \dots, x_n) = Q(x_1, \dots, x_n)\\). Equivalently, is \\(P - Q\\) the zero polynomial? If you have \\(P\\) and \\(Q\\) only as black boxes that compute outputs given inputs, you cannot inspect their structure — but you can evaluate \\(P - Q\\) at a random point. If non-zero, not equal. If zero, probably equal.

This is *the* nontrivial example in the literature on randomized algorithms versus deterministic ones. It has been an open question for decades whether polynomial identity testing has a polynomial-time deterministic algorithm. A deterministic algorithm would have profound consequences (it would imply circuit lower bounds nobody knows how to prove), so most researchers believe it is hard. The randomized version is one line.

## Bipartite matching

Tutte's theorem says: a bipartite graph \\(G\\) with vertex sets \\(\{u_1, \dots, u_n\\}\\) and \\(\{v_1, \dots, v_n\\}\\) has a perfect matching if and only if the determinant of a certain symbolic matrix is non-zero. The matrix \\(A\\) has entry

\\[ A_{ij} = \begin{cases} x_{ij} & \text{if } u_i v_j \text{ is an edge} \\ 0 & \text{otherwise} \end{cases} \\]

where the \\(x_{ij}\\) are formal variables. The determinant \\(\det(A)\\) is a polynomial in the \\(x_{ij}\\). Each non-zero term of the determinant corresponds to a perfect matching (which permutation \\(\sigma\\) of \\(\{1, \dots, n\\}\\) you use), and the term is \\(\pm \prod x_{i, \sigma(i)}\\). Different matchings produce different monomials, so they cannot cancel. Hence \\(\det(A) \neq 0\\) as a polynomial iff a perfect matching exists.

But \\(\det(A)\\) is exponentially large — \\(n!\\) terms. We cannot compute it symbolically.

Schwartz–Zippel: pick random integers in some range, plug them in for the \\(x_{ij}\\), compute the resulting numerical determinant in \\(O(n^3)\\) (or \\(O(n^\omega)\\) with fast matrix multiplication). If non-zero, a matching exists. If zero, probably no matching, with quantifiable error.

This gives a parallelizable algorithm for bipartite matching that fits in \\(\text{RNC}\\) (randomized fast parallel time). The deterministic version is also polynomial but much more complicated; Schwartz–Zippel hands you the parallel version for free.

## Tree isomorphism, polynomial-time

Given two rooted labeled trees, are they the same tree (up to relabeling of children at each level)? You could write a recursive canonical-form algorithm. Or you could associate each tree with a polynomial: define a polynomial recursively over the tree, then use Schwartz–Zippel to test polynomial equality.

For each leaf, the polynomial is some fixed value. For each internal node with children whose polynomials are \\(p_1, \dots, p_k\\), the node's polynomial is \\(\prod (x_d - p_i)\\) where \\(d\\) is the depth. Equal trees produce identical polynomial. Different trees produce different polynomials (with high probability under random plugging).

This is a clean illustration of the meta-trick: encode a discrete structure as a polynomial whose vanishing detects structural equality, then test the polynomial.

## SNARKs and PCPs

In zero-knowledge proofs and probabilistically checkable proofs, the verifier wants to check that a prover's claimed solution is correct, by examining only a few random bits. The arithmetization step turns the computation into a polynomial whose vanishing on a designated set encodes correctness. The verifier evaluates the polynomial at a random point. By Schwartz–Zippel, if the prover's polynomial differs from the true one (i.e., the prover is cheating), the random evaluation catches it with high probability.

The whole zk-SNARK toolchain rests on this. Without Schwartz–Zippel, sublinear-verifier proof systems would not exist.

## The wonder

Most algorithmic problems in algebra come with a depressing computational floor: simplifying a symbolic expression takes exponential time in general; comparing two such expressions, even more. Schwartz–Zippel gives you a randomized shortcut that beats the floor, by exploiting the fact that a polynomial with low degree is *forced* to be small over the variety of its vanishing points. Plug in a random point and you are almost certainly off the variety, in which case you see the polynomial's true value.

It is a perfect example of how randomness, carefully deployed, lets you avoid doing work the algebraic structure of the problem made expensive. The randomness is not a heuristic. It is a probability-1-minus-epsilon proof, with the epsilon under your control.

## Where to go deeper

- Mulmuley, Vazirani, Vazirani, *Matching is as Easy as Matrix Inversion*, STOC 1987. The bipartite-matching trick.
- Motwani and Raghavan, *Randomized Algorithms*, Chapter 7. Standard reference, with the proof and several applications worked out.
