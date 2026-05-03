# The arithmetical hierarchy

Some statements about natural numbers can be checked by an algorithm. "Is \\(n\\) prime?" — a Turing machine answers yes or no in finite time. "Does \\(P\\) halt on input \\(I\\)?" — undecidable, but answered yes by a *partial* algorithm that simulates and waits. "Does there exist a \\(P\\) that halts on every input?" — even harder. As you stack quantifiers (∀∃, ∃∀∃, …) over arithmetic predicates, you climb a *hierarchy of difficulty*, and the levels of the hierarchy are provably distinct.

This is the arithmetical hierarchy. Each level is strictly more powerful than the one below. The undecidability of the halting problem is the first non-trivial step, and the hierarchy continues forever above it. Computability theory uses it to classify problems by how many quantifier alternations are needed to define them.

## The setup

A *computable* (recursive) predicate is one decidable by an algorithm: \\(R(x)\\) is computable iff some Turing machine, on input \\(x\\), eventually outputs "yes" or "no" depending on whether \\(R(x)\\) holds.

A *recursively enumerable (r.e.)* set is the *halting set* of some Turing machine: it is the set of inputs on which the machine halts with "accept." Equivalently, the set of \\(x\\) such that some computable predicate \\(R(x, y)\\) holds for some \\(y\\):

\\[ x \in S \iff \exists y. \, R(x, y) \\]

So r.e. sets are *one-quantifier-existential* over computable predicates. The halting problem is the canonical r.e. but undecidable problem: \\(\text{HALT} = \{(P, I) : \exists t. \, P \text{ halts on } I \text{ in } t \text{ steps}\}\\).

A set is *co-r.e.* if its complement is r.e.: \\(x \in S \iff \forall y. \, R(x, y)\\), one-quantifier-universal.

## The levels

\\(\Sigma^0_0 = \Pi^0_0 = \Delta^0_1\\): computable predicates.

\\(\Sigma^0_1\\): predicates of the form \\(\exists y. \, R(x, y)\\) for \\(R\\) computable. r.e. sets.

\\(\Pi^0_1\\): \\(\forall y. \, R(x, y)\\). Co-r.e. sets.

\\(\Sigma^0_2\\): \\(\exists y_1 \, \forall y_2. \, R(x, y_1, y_2)\\). Two quantifier alternations starting with \\(\exists\\).

\\(\Pi^0_2\\): \\(\forall y_1 \, \exists y_2. \, R(x, y_1, y_2)\\). Two quantifiers starting with \\(\forall\\).

In general, \\(\Sigma^0_n\\) is \\(n\\) alternations starting with \\(\exists\\); \\(\Pi^0_n\\) is \\(n\\) starting with \\(\forall\\). The hierarchy goes:

```
                 ...
                  |
              Δ^0_3 = Σ^0_3 ∩ Π^0_3
              /                 \\
        Σ^0_2                    Π^0_2
              \\                 /
              Δ^0_2 = Σ^0_2 ∩ Π^0_2
              /                 \\
        Σ^0_1                    Π^0_1     <- r.e. and co-r.e.
              \\                 /
              Δ^0_1 = computable predicates
```

Each \\(\Sigma^0_n \subsetneq \Sigma^0_{n+1}\\) (strict). \\(\Sigma^0_n\\) and \\(\Pi^0_n\\) are incomparable but their intersection \\(\Delta^0_{n+1}\\) is strictly between \\(\Sigma^0_n\\) (or \\(\Pi^0_n\\)) and \\(\Sigma^0_{n+1}\\).

## Examples

**\\(\Sigma^0_1\\)**: "\\(P\\) halts on input \\(I\\)" — \\(\exists t. (P \text{ halts in } t)\\). The halting set is r.e.

**\\(\Pi^0_1\\)**: "\\(P\\) halts on no input" (the *totality of non-halting*) — \\(\forall I. \neg \text{Halt}(P, I)\\). Co-r.e.

**\\(\Pi^0_2\\)**: "\\(P\\) is total" — \\(\forall I \, \exists t. \text{Halt}(P, I, t)\\). \\(P\\) halts on every input.

**\\(\Sigma^0_2\\)**: "There exists \\(I\\) on which \\(P\\) halts in finitely many steps for every \\(t\\)..." (slightly contrived but possible).

**\\(\Pi^0_3\\)**: "\\(P\\) is total *and* runs in time \\(O(n^k)\\) for some \\(k\\)" — \\(\exists k \, \forall I \, \exists t \leq f(|I|, k). \text{Halt}(P, I, t)\\), with appropriate bounds.

**Cofinite halting**: "\\(P\\) halts on all but finitely many inputs" — \\(\Sigma^0_3\\).

The Goldbach conjecture is in \\(\Pi^0_1\\): "\\(\forall n \geq 4. \, n \text{ is even} \to \exists p, q \text{ prime with } p + q = n\\)" — universal in \\(n\\), with bounded existentials inside, which makes the inner part computable.

## Why the levels are distinct

Each level has a *complete* problem — one to which all problems at that level can be reduced. The completeness of \\(\Sigma^0_n\\) for the next-higher quantifier alternation can be proved by diagonalization (see *Cantor diagonalization*). So \\(\Sigma^0_n \neq \Sigma^0_{n+1}\\) for every \\(n\\); the hierarchy is strict.

This is essentially the same diagonal argument as for the halting problem, applied at each level. At level \\(n\\), you have an enumeration of \\(\Sigma^0_n\\) sets; you construct a set that differs from each in some specifically-chosen point; the new set is \\(\Sigma^0_{n+1}\\) but not \\(\Sigma^0_n\\).

## What this classifies

Pretty much every interesting problem about computation lives somewhere in the hierarchy:

- **Halting problem** \\(\text{HALT}\\): \\(\Sigma^0_1\\)-complete.
- **Total halting** \\(\text{TOT}\\) (does \\(P\\) halt on every input?): \\(\Pi^0_2\\)-complete.
- **Recursive equivalence** (do \\(P, Q\\) compute the same function?): \\(\Pi^0_2\\)-complete.
- **Cofiniteness of halting domain** (does \\(P\\) halt on all but finitely many inputs?): \\(\Sigma^0_3\\)-complete.
- **\\(P\\) computes a recursive function**: \\(\Sigma^0_3\\)-complete.
- **\\(P\\) computes a primitive-recursive function**: \\(\Sigma^0_4\\) at least.

The further you go up the hierarchy, the more "self-referential" or "introspective" the question. Level 1 is "does \\(P\\) halt?" Level 2 is "is \\(P\\) total?" Level 3 is "does \\(P\\) compute *some* recursive function?"

## Beyond the arithmetical hierarchy

Past \\(\Sigma^0_\omega\\) (the union of all finite levels), one continues into the *analytical hierarchy* \\(\Sigma^1_n, \Pi^1_n\\), which uses second-order quantification (over functions, not just numbers). Set theory at this level connects to descriptive set theory and large-cardinal axioms.

Above that: the projective hierarchy, the constructible hierarchy (\\(L_\alpha\\)), the cumulative hierarchy of sets. Each level of complexity has its own characterizing axioms and consistency strength.

## Why this matters

The arithmetical hierarchy is the right framework for thinking about *unsolvability*. "Undecidable" is too coarse; many undecidable problems are not equivalent to each other. The halting problem and the totality problem are both undecidable, but solving one would not necessarily let you solve the other (you would need an oracle for \\(\Sigma^0_1\\) for halting, and for \\(\Pi^0_2\\) for totality — different oracles).

Computability theory works mostly with the hierarchy. Many results in classical computability ("Rice's theorem says any non-trivial property of programs is undecidable") are sharpened by saying *exactly* where each property lives in the hierarchy.

It is also the framework underneath various consistency-strength results in proof theory: "system \\(T\\) proves the consistency of system \\(S\\)" means roughly "\\(T\\) can prove a \\(\Pi^0_1\\) statement (consistency of \\(S\\)) that \\(S\\) cannot prove."

## The wonder

There is a strict, infinite tower of "harder than computable" problems. Each level is provably distinct, populated by natural problems, and characterized by quantifier alternation. The levels are not artificial: real questions about programs (halting, totality, equivalence, density) sit at definite levels.

The wonder is in the strictness. You might hope that "harder than computable" was a single category — there is the halting problem, and everything else is similar. The arithmetical hierarchy says no: there are *infinitely many strictly harder problems*, organized in a well-defined ladder, each level provably more difficult than the last. And the ladder, as deep as it is, is just the start; the analytical hierarchy and beyond extend further.

The hierarchy gives a quantitative answer to "how hard is your problem?" — at least for problems involving natural numbers. For problems beyond arithmetic (set theory, real analysis), there are similar but more elaborate hierarchies. In every case, the structure is the same: quantifier alternation gives strictly more expressive power, indefinitely.

## Where to go deeper

- Soare, *Recursively Enumerable Sets and Degrees* (Springer, 1987). The classical computability theory text.
- Cooper, *Computability Theory* (Chapman and Hall, 2003). Modern textbook with the hierarchy and degree-theoretic results.
