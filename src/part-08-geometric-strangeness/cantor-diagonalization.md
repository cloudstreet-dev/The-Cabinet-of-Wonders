# Cantor diagonalization

There are more real numbers than there are integers. Strictly more. Not just "more in some intuitive sense" but more in the formal sense that no list — no infinite enumeration — of real numbers can include all of them. Cantor proved this in 1891 with one of the most economical arguments in mathematics. The same proof technique then opens out to give the halting problem, Gödel's incompleteness theorem, and a dozen other "cannot do this in general" results.

The argument is half a page. It is a wonder both because it works and because it works *everywhere*, in a thousand different mathematical contexts, by the same essential move.

## The proof

Suppose, for contradiction, that the real numbers in \\([0, 1)\\) can be listed:

\\[ r_1 = 0.d_{11} d_{12} d_{13} d_{14} \dots \\]
\\[ r_2 = 0.d_{21} d_{22} d_{23} d_{24} \dots \\]
\\[ r_3 = 0.d_{31} d_{32} d_{33} d_{34} \dots \\]
\\[ \vdots \\]

where each \\(d_{ij}\\) is a decimal digit. The list is allegedly complete: every real in \\([0, 1)\\) appears as some \\(r_n\\).

Construct a new real \\(x = 0. e_1 e_2 e_3 \dots\\) where \\(e_i\\) is chosen to *differ from* \\(d_{ii}\\). For concreteness: \\(e_i = 5\\) if \\(d_{ii} \neq 5\\), else \\(e_i = 6\\).

Now \\(x\\) is in \\([0, 1)\\). Is it in the list? It cannot be \\(r_1\\), because they differ in the first decimal place. It cannot be \\(r_2\\), because they differ in the second decimal place. In general, \\(x \neq r_n\\) for every \\(n\\) because they differ in the \\(n\\)-th decimal.

So \\(x\\) is a real number not in the list. Contradiction. The list cannot be complete.

(There is a small technical issue with decimals like 0.4999... = 0.5000... having two representations, handled by avoiding 0 and 9 in the construction. The argument is robust.)

## What it shows

\\(\mathbb{R}\\) has *strictly larger cardinality* than \\(\mathbb{N}\\). The set of all subsets of \\(\mathbb{N}\\) (the power set) has the same cardinality as \\(\mathbb{R}\\); both are \\(2^{\aleph_0}\\), the cardinality of the continuum.

The same argument generalizes: for any set \\(S\\), the power set \\(\mathcal{P}(S)\\) has strictly larger cardinality than \\(S\\). Cantor's theorem: \\(|S| < |\mathcal{P}(S)|\\) for every set.

So there is a strictly increasing tower of infinities: \\(\aleph_0 < 2^{\aleph_0} < 2^{2^{\aleph_0}} < \dots\\). No largest cardinality exists.

## The diagonal as a method

The trick generalizes far beyond cardinalities. The recipe:

1. Suppose every \\(X\\) of some kind is enumerable as \\(X_1, X_2, \dots\\).
2. Construct a new \\(X^*\\) whose \\(n\\)-th feature differs from \\(X_n\\)'s \\(n\\)-th feature.
3. \\(X^*\\) is of the same kind, but cannot equal any \\(X_n\\).
4. Contradiction.

This is the diagonal method. Applications:

**Halting problem.** Suppose there is an algorithm \\(H\\) that decides, for any program \\(P\\) and input \\(I\\), whether \\(P\\) halts on \\(I\\). Construct a new program \\(D\\) that, given input \\(P\\), runs \\(H(P, P)\\); if \\(H\\) says "halts," go into an infinite loop; if "doesn't halt," halt immediately. What does \\(H(D, D)\\) say? Either answer leads to a contradiction. So \\(H\\) cannot exist.

\\(D\\)'s behavior on input \\(D\\) is the *diagonal element*: \\(D\\) defined to disagree with itself. Diagonalization proves the halting problem is undecidable.

**Gödel's incompleteness.** Construct a sentence that says "I am not provable in this formal system." If the system is consistent and the sentence is provable, the sentence is true (so it's also unprovable, contradiction). If unprovable, the sentence is true but unprovable — incompleteness. The construction of the self-referential sentence uses a diagonalization-by-Gödel-numbering trick (see the *Gödel's coding trick* entry).

**Russell's paradox.** Define \\(R = \{x : x \notin x\}\\). Is \\(R \in R\\)? Either answer contradicts. The contradiction is exactly Cantor's diagonal applied to "the set of all sets containing themselves."

**Tarski's undefinability of truth.** Truth in arithmetic is not definable inside arithmetic. Same diagonal move.

**Yablo's paradox** (no self-reference, but a sequence of statements each saying "all the later ones are false"). The diagonal is implicit but generalizes the technique.

## Why it always works

The structural feature: any system that lets you describe its own elements (as bit strings, programs, sentences, numbers) and lets you *negate* (flip) those descriptions can be diagonalized. The negation is local; the description is uniform; so a self-diagonal element exists, and it must contradict its own listing.

This is essentially the entire content of "you cannot enumerate everything that interacts with itself." The diagonal element is constructed to disagree with the alleged enumeration in exactly the place that names it. The contradiction is unavoidable given the assumed structure.

## Beyond mathematics

Diagonalization shows up in:

- **Computational complexity**: the time hierarchy theorem (more time gives strictly more computational power) is proved by a diagonal argument: for each time bound, exhibit a problem whose solver requires more time.
- **Kolmogorov complexity**: \\(K\\) is uncomputable because the assumption that it is computable lets you build a paradoxical program (see *Kolmogorov complexity*).
- **Type theory**: Russell's paradox motivates type stratification. Universe hierarchies in modern type theories (Coq, Lean) are designed to block the diagonalization.
- **Tarski's "set of true sentences" cannot be defined inside the language**.

## The wonder

A single technique — define an entity to disagree with the diagonal of any alleged enumeration — knocks out an entire family of "you can list everything" claims. From cardinalities of sets to decidability of programs to provability of arithmetic, the same move applies, and the conclusion is: you cannot list it all, you cannot decide it all, you cannot prove it all. There are too many of *something*, in a precise sense.

The wonder is in the universality. Cantor's argument was about real numbers; once Turing and Gödel saw it, they realized it could be retold for programs and proofs, with the same conclusion. The 1891 paper is a foundational seed of every undecidability result of the next century. Each new application is a re-execution of the same diagonal step in a new costume.

The diagonal is the thing in mathematics most resembling a master key. Once you have it, you can open a remarkable number of impossibility-of-listing doors.

## Where to go deeper

- Cantor, *Über eine elementare Frage der Mannigfaltigkeitslehre*, 1891. The original.
- Smullyan, *Diagonalization and Self-Reference* (Oxford, 1994). The systematic survey of the technique's applications.
