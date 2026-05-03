# Kolmogorov complexity

The amount of information in a string is the length of the shortest computer program that prints it. This sounds like a hand-wavy slogan but it is precise mathematics, with a startling consequence: information content is *uncomputable*. There is no algorithm that takes a string as input and returns its Kolmogorov complexity. Yet the quantity is well-defined, and reasoning about it lets you prove things you could not prove any other way.

Kolmogorov, Solomonoff, and Chaitin independently arrived at this in the 1960s. It is the cleanest formal definition of "information" in the algorithmic sense, complementing Shannon's statistical entropy.

## The definition

Fix a universal Turing machine \\(U\\). The Kolmogorov complexity \\(K(s)\\) of a string \\(s\\) is the length of the shortest program \\(p\\) such that \\(U(p) = s\\):

\\[ K(s) = \min \{ |p| : U(p) = s \} \\]

Different choices of \\(U\\) give different \\(K\\), but only by an additive constant: if \\(U_1\\) and \\(U_2\\) are two universal machines, then \\(K_{U_1}(s) \leq K_{U_2}(s) + c_{12}\\) for a constant \\(c_{12}\\) (the length of an interpreter for \\(U_2\\) running on \\(U_1\\)). So \\(K\\) is well-defined up to a fixed additive constant, which becomes negligible for long strings.

Examples:

- \\(K(\text{"AAAAAAA...A"}, 1{,}000{,}000) = O(\log(10^6))\\): a short program "print 'A' a million times" generates it.
- \\(K(\pi)\\) up to the millionth digit: \\(O(\log 10^6)\\): a short program computes \\(\pi\\) by series and prints.
- \\(K\\) of a uniformly random binary string of length \\(n\\): \\(\approx n\\) — almost incompressible, with high probability.

A string is *random* in the algorithmic sense (or *Kolmogorov-random*) if \\(K(s) \approx |s|\\). Almost all strings are random; very few are compressible.

## Why it is uncomputable

Suppose for contradiction there exists a computable function \\(f(s) = K(s)\\). Define the program: "Find the lexicographically first string \\(s\\) with \\(K(s) > N\\); print \\(s\\)." This program has length about \\(\log N + c\\). It outputs a string with complexity greater than \\(N\\). But we just exhibited a program of length \\(\log N + c\\) that outputs that string. So \\(K(s) \leq \log N + c < N\\). Contradiction for large \\(N\\).

This is *Berry's paradox* turned into a theorem. The paradox ("the smallest positive integer not definable in fewer than twelve words") trades on a self-referential vagueness; making the definition computable removes the vagueness and gives a real impossibility.

## Why it matters anyway

Even though you cannot compute \\(K\\), you can:

**Prove lower bounds**. If you want to show no algorithm can do task \\(T\\) faster than \\(f(n)\\), you can sometimes show that fast algorithms would let you compute \\(K\\) on too many inputs, contradicting incompressibility. The "incompressibility method" is a powerful proof technique in computational complexity (Li-Vitanyi).

**Define randomness rigorously.** A string is "random" iff its Kolmogorov complexity is close to its length. This is the algorithmic definition of randomness, complementing the statistical definition. They mostly coincide on long strings but diverge in subtle cases.

**Define a universal prior.** The Solomonoff prior \\(P(s) = 2^{-K(s)}\\) (suitably normalized) is a probability distribution that assigns probability to strings inversely proportional to their algorithmic complexity. It is a kind of "universal" Occam's razor: simpler hypotheses (shorter programs) are more probable. Solomonoff's prior is also uncomputable, but provides a theoretical optimum for inductive inference: a Bayes-optimal predictor using the universal prior would, in the limit, learn anything learnable.

**Prove information-theoretic facts that resemble Shannon's, in the algorithmic regime.** \\(K(s, t) \leq K(s) + K(t | s) + O(\log K(s, t))\\) — chain rule. \\(K(s) - K(s | t)\\) — algorithmic mutual information. The whole edifice of Shannon information theory has an algorithmic counterpart with similar identities.

## The Chaitin constant

A specific uncomputable real number: \\(\Omega\\), the *Chaitin constant*, is the probability that a randomly generated binary program halts on a fixed prefix-free universal Turing machine.

\\(\Omega\\) is well-defined as a real number in (0, 1). It is uncomputable in the strongest sense: knowing the first \\(n\\) bits of \\(\Omega\\) lets you decide the halting problem for all programs of length up to \\(n\\). So \\(\Omega\\) is *algorithmically random* — its bits are incompressible in the Kolmogorov sense.

Chaitin used \\(\Omega\\) to prove a quantitative version of Gödel's theorem: any formal axiomatic system with computable axioms can prove only finitely many bits of \\(\Omega\\). There are statements about specific bits of \\(\Omega\\) ("the 1729th bit is 1") that are independent of any reasonable axiom system — undecidable not for foundational reasons, but for *information-theoretic* reasons. The axiom system is a finite object; \\(\Omega\\) contains infinite information; you cannot extract more bits of information from a finite axiom system than the system contains.

## Compressed strings, in practice

The relationship to actual compressors: if a compressor outputs a representation of \\(s\\) of length \\(L(s)\\), then \\(K(s) \leq L(s) + O(1)\\) (the constant being the size of the decompressor). So practical compressors give upper bounds on Kolmogorov complexity.

This gives a heuristic notion of "approximate \\(K\\)" using gzip or similar: the compressed length of a string is a (loose) upper bound on its algorithmic complexity. Used in *normalized compression distance* (NCD), a practical metric for comparing strings or files: \\(\text{NCD}(x, y) = \frac{K(xy) - \min(K(x), K(y))}{\max(K(x), K(y))}\\), approximated by \\(C(xy) - \min(C(x), C(y))\\) over the max.

NCD is used for clustering DNA sequences, classifying languages, plagiarism detection, and a few other tasks where similarity-of-information is what you want.

## The wonder

Kolmogorov complexity formalizes what "information content" means in a way that does not depend on probability. A string has information content equal to the length of the shortest program for it. Probabilistic strings (uniformly random) and deterministic strings (\\(\pi\\)'s digits, the prime sequence, the natural numbers) both fit in the same framework: the random ones are incompressible, and the deterministic-but-not-random ones (where there exists a short generating program, even though they look complicated) are compressible.

The uncomputability is an honest part of the story. There is no shortcut to knowing the exact information content of a string. You can prove upper bounds (with compressors) and lower bounds (with the incompressibility method, by exhibiting consequences that would follow from too-low complexity). The actual quantity sits behind a veil. But it is well-defined, and reasoning about it gives some of the cleanest proofs in computer science — proofs that random behavior is forced by counting (because most strings are random) and that complexity is hereditary (compressed sub-strings of an incompressible string would compress the whole, contradiction).

The wonder is that information itself, in the algorithmic sense, is provably uncomputable. We have a perfectly good definition, and the definition has an inherent obstruction: any algorithm to compute it would have to be more powerful than the universal Turing machine. The undecidability of \\(K\\) is just the halting problem in another costume.

## Where to go deeper

- Li and Vitanyi, *An Introduction to Kolmogorov Complexity and Its Applications*. The textbook. Read Chapters 1-3.
- Chaitin, *The Unknowable* (1999). Popular but technical, accessible introduction with original results.
