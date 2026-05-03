# Curry–Howard correspondence

A program is a proof. A proof is a program. The two are literally the same thing, written in different notations. Each typed program corresponds to a logical proposition (its type) and a proof of that proposition (the program itself). Each logical proof corresponds to a function. Compose two proofs by feeding the output of one into the input of the other; you have just composed two functions.

This is the Curry-Howard correspondence. It was noticed in pieces over decades — Howard's 1969 manuscript, Curry's 1934 work on combinatory logic — and it underlies every modern proof assistant (Coq, Agda, Lean, Idris) and a great deal of programming language theory.

The wonder is double. First, that two seemingly distinct disciplines (logic and computation) turn out to be two faces of the same object. Second, that this discovery has practical consequences: type-checking a program is the same operation as verifying a proof, so a sufficiently expressive type system can express and check arbitrary mathematical proofs.

## The dictionary

The correspondence assigns to each logical construct a programming construct:

| Logic | Programming |
|---|---|
| Proposition | Type |
| Proof of \\(P\\) | Term of type \\(P\\) |
| Implication \\(P \to Q\\) | Function type \\(P \to Q\\) |
| Conjunction \\(P \land Q\\) | Pair (product) type \\(P \times Q\\) |
| Disjunction \\(P \lor Q\\) | Sum (variant) type \\(P + Q\\) |
| True (trivially provable) | Unit type \\(\mathbf{1}\\) (one inhabitant) |
| False (no proof) | Empty type \\(\mathbf{0}\\) (no inhabitants) |
| Negation \\(\neg P\\) | \\(P \to \mathbf{0}\\) (function from \\(P\\) to absurd) |
| Universal \\(\forall x. P(x)\\) | Dependent product \\(\Pi x : A. P(x)\\) |
| Existential \\(\exists x. P(x)\\) | Dependent sum \\(\Sigma x : A. P(x)\\) |

Each row is a strict equivalence — not analogy, identity. The proof rules of natural deduction become the typing rules of a lambda calculus. *Modus ponens* (from \\(P\\) and \\(P \to Q\\), conclude \\(Q\\)) is function application: from \\(p : P\\) and \\(f : P \to Q\\), get \\(f(p) : Q\\).

## A worked example

Logical proposition: \\(P \to (Q \to P)\\). "If \\(P\\), then \\(Q\\) implies \\(P\\)."

Proof: assume \\(p : P\\). Now in the inner scope, assume \\(q : Q\\). The conclusion is \\(P\\), and we have \\(p : P\\). Discharge \\(q\\): \\(\lambda q. p : Q \to P\\). Discharge \\(p\\): \\(\lambda p. \lambda q. p : P \to (Q \to P)\\).

Programming version: write the function \\(\lambda p. \lambda q. p\\). This is the K combinator from combinatory logic, the Haskell `const` function. Its type is \\(P \to Q \to P\\). Type-checking it confirms it has that type, which is the same as confirming it proves \\(P \to (Q \to P)\\).

The proof and the program are literally the same object — depending on whether you read the lambda calculus expression as code or as a proof tree.

## Computation = proof normalization

In logic, a proof can be *normalized*: redundant detours are eliminated. For instance, if you proved \\(P\\), then derived \\(P \to Q\\) by some chain, then concluded \\(Q\\) by modus ponens, the *normalized* proof might just be a direct proof of \\(Q\\).

In programming, a function applied to an argument *reduces* by beta-reduction: \\((\lambda x. e)(v) \to e[v/x]\\). Reducing a program to a normal form is the program execution (or evaluation).

These are the same operation. Normalizing a proof = evaluating the corresponding program. The cut-elimination theorem of logic (every proof can be normalized) is the strong normalization theorem of typed lambda calculus (every typed expression reduces to a value).

So *running a program is reducing a proof to a value*. Every type-checked program has a corresponding proposition (its type), the program is a proof of that proposition, and executing the program is mechanically simplifying the proof to its essential form.

## What this enables: dependent types

Simple type theories (like Haskell's) restrict types to first-order objects. Dependent type theories let types depend on values, which corresponds to allowing predicates that depend on terms — i.e., quantified statements.

In dependent type theory, you can write a function whose type is

\\[ \Pi n : \mathbb{N}. \, \text{IsSorted}(\text{sort}(n)) \\]

This type says: for any natural number \\(n\\), the result of `sort(n)` is sorted. A function inhabiting this type is, simultaneously, a sorting algorithm *and a proof that the algorithm produces sorted output*. The type-checker verifies the proof.

This is what proof assistants like Coq, Agda, and Lean do. They are programming languages whose type system is so rich that you can express mathematical theorems as types. A program of that type is a proof of the theorem. Type-checking is proof verification.

## Constructive logic and computational interpretation

Curry-Howard works for *constructive* (intuitionistic) logic, not classical. Classical logic includes the law of excluded middle (\\(P \lor \neg P\\) for all \\(P\\)), which has no straightforward computational interpretation: a proof of \\(P \lor \neg P\\) ought to either provide a \\(P\\) or a refutation, but excluded middle doesn't say which.

Adding excluded middle to the constructive logic corresponds, on the programming side, to adding a *control operator* like `call/cc`. So classical proofs become programs that may invoke continuations. (Griffin 1990 showed this explicitly.)

This was a genuinely surprising consequence of taking Curry-Howard seriously: classical logic, often considered the natural setting for mathematics, corresponds to a programming language with first-class continuations, which is more exotic than the constructive case.

## Where it shows up in practice

- **Coq, Agda, Lean, Idris**: programming languages whose type-checker is also a proof assistant. Used for verifying critical software (CompCert C compiler, seL4 microkernel) and major mathematics (Four-Color Theorem in Coq, Liquid Tensor Experiment in Lean, formalizations of perfectoid spaces).
- **Haskell, OCaml, Rust** (in spirit): even non-dependent type systems implement parts of Curry-Howard. A Haskell function of type `(a -> b) -> ([a] -> [b])` is a proof of "if I have a function from \\(A\\) to \\(B\\), I have a function from list-of-\\(A\\) to list-of-\\(B\\)" — the proposition that lists are functorial.
- **Type-driven development**: writing types first, then realizing the types' meaning forces certain code structures.
- **Theory of programming languages**: every PL theorist works in this framework.

## What it predicts that monads do

In Haskell, monads are a way to organize sequenced computations with side effects. In Curry-Howard terms, monads are *modal logics*: \\(M(P)\\) is the proposition "I can do an effect to produce a \\(P\\)." The monadic bind \\(\geq\!\!=: M(P) \to (P \to M(Q)) \to M(Q)\\) corresponds to a modal axiom.

This connection deepens the formalism: lots of "advanced" programming patterns (monads, applicatives, comonads, indexed monads) correspond to known modal logics, which have been studied in philosophy and computer science for decades.

## The wonder

A program is a proof. A proof is a program. They are not metaphors for each other. They are literally the same object viewed in two equivalent ways. The lambda term \\(\lambda x. \lambda y. x\\) is, depending on how you label it, the constant function or the proof of \\(A \to B \to A\\). The substitution rule that drives proof normalization is the beta-reduction rule that drives program execution. The type-checker that verifies a program's type is a proof-checker for the corresponding theorem.

This is one of those identifications that, once made, becomes obvious in hindsight but was not seen for half a century. Logic and programming developed mostly in parallel through the early 20th century; the equivalence was clarified by Howard in 1969, with bits in earlier work, and only became foundational with the development of dependent type theories in the 1970s and 1980s.

The practical impact is enormous. Modern proof assistants are programming languages. Modern programming languages are getting more proof-aware. Software verification, mathematics formalization, and programming-language design are converging on the same toolset, and that toolset is just the Curry-Howard correspondence taken seriously.

## Where to go deeper

- Wadler, *Propositions as Types*, CACM 2015. The short, beautiful overview.
- Sørensen and Urzyczyn, *Lectures on the Curry-Howard Isomorphism* (Elsevier, 2006). Textbook, comprehensive.
