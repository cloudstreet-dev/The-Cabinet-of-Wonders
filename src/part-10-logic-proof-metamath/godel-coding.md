# Gödel's coding trick

To prove that arithmetic is incomplete — that some statements about whole numbers are true but unprovable from any reasonable set of axioms — Gödel needed arithmetic to *talk about itself*. He needed sentences of arithmetic that referred to other sentences, including their own existence and their own provability. Arithmetic is a language for talking about numbers, not about sentences. So Gödel made arithmetic talk about sentences by *encoding sentences as numbers*.

The encoding is a recipe for assigning a unique natural number to each sentence (or proof, or any finite string of symbols) of a formal system. Once you have it, statements about sentences become statements about numbers, and statements about numbers can be made within arithmetic itself. Then you can write down the sentence "there is no number coding a proof of *this very sentence*" — a sentence that says it is unprovable. If unprovable, true; if provable, false (so the system is unsound). Either way, incompleteness.

The trick is in the encoding. Once you have it, the rest of the proof is bookkeeping.

## The encoding

Assign to each *symbol* of the formal language a small natural number — say:

- ¬ → 1
- ∨ → 2
- ∀ → 3
- 0 → 4
- s → 5 (successor)
- ( → 6
- ) → 7
- variables \\(v_0, v_1, v_2, \dots\\) → 8, 9, 10, …

To encode a *string* of symbols \\(s_1 s_2 \dots s_n\\) (where each \\(s_i\\) is a symbol with code \\(c_i\\)), use the prime factorization trick:

\\[ \text{Gödel number}(s_1 s_2 \dots s_n) = 2^{c_1} \cdot 3^{c_2} \cdot 5^{c_3} \cdot 7^{c_4} \cdots p_n^{c_n} \\]

where \\(p_n\\) is the \\(n\\)-th prime. By unique prime factorization, the Gödel number uniquely determines the string. Distinct strings get distinct numbers.

Encode *sequences of strings* (for proofs, which are sequences of formulas) by analogously taking primes-to-Gödel-numbers:

\\[ \text{Gödel number}(\text{proof of } n \text{ steps}) = 2^{g_1} \cdot 3^{g_2} \cdots p_n^{g_n} \\]

where \\(g_i\\) is the Gödel number of the \\(i\\)-th formula in the proof.

So every sentence of arithmetic has a Gödel number; every proof has a Gödel number.

## The arithmetization

Now you can talk about sentences within arithmetic. Examples of arithmetic predicates that decode the Gödel-numbering:

- \\(\text{IsFormula}(x)\\): "\\(x\\) is the Gödel number of a well-formed formula." A primitive-recursive predicate, expressible in arithmetic.
- \\(\text{IsAxiom}(x)\\): "\\(x\\) is the Gödel number of an axiom." Also primitive-recursive.
- \\(\text{IsProof}(p, x)\\): "\\(p\\) is the Gödel number of a valid proof of the formula with Gödel number \\(x\\)." Primitive-recursive.
- \\(\text{Provable}(x)\\) := \\(\exists p. \, \text{IsProof}(p, x)\\): "\\(x\\) is the Gödel number of a provable formula." Recursively enumerable; not primitive-recursive (proofs can be arbitrarily long).

These are arithmetic statements — they say things about the natural numbers (specifically, about Gödel numbers, but those are just numbers).

## The diagonal lemma

The next move is the diagonal lemma (also called the *fixed-point lemma*): for any arithmetic formula \\(\phi(x)\\) with one free variable, there is a sentence \\(G\\) (closed formula) such that

\\[ G \leftrightarrow \phi(\ulcorner G \urcorner) \\]

where \\(\ulcorner G \urcorner\\) denotes the Gödel number of \\(G\\). In words: there is a sentence that says \\(\phi\\) holds of itself.

The proof of the diagonal lemma is a clever construction (using a *substitution function* \\(\sigma\\) that takes a Gödel number of a formula \\(\phi(x)\\) and returns the Gödel number of \\(\phi(\ulcorner \phi \urcorner)\\)). It is the core trick of self-reference, made formal.

## Gödel's sentence

Apply the diagonal lemma to \\(\phi(x) := \neg \text{Provable}(x)\\). You get a sentence \\(G\\) such that

\\[ G \leftrightarrow \neg \text{Provable}(\ulcorner G \urcorner) \\]

\\(G\\) says: "I am not provable."

Now reason about \\(G\\):

- If \\(G\\) is provable, then \\(\text{Provable}(\ulcorner G \urcorner)\\) is true. By \\(G\\)'s definition, \\(\neg G\\) is also true. So \\(\neg G\\) and \\(G\\) are both provable; the system is *inconsistent*. Assuming consistency, \\(G\\) is not provable.
- Since \\(G\\) is not provable, \\(\neg \text{Provable}(\ulcorner G \urcorner)\\) is true. By \\(G\\)'s equivalence, \\(G\\) is true.

So \\(G\\) is true (in the standard model of arithmetic) and unprovable. This is *Gödel's first incompleteness theorem*: any consistent recursively-axiomatized theory containing arithmetic has true statements that are unprovable in it.

## What the coding trick really does

The genius is not the diagonal lemma alone, or the construction of \\(G\\). It is the *embedding of syntax into semantics*. Gödel made the syntax of arithmetic (the formulas) into objects of arithmetic (the Gödel numbers). Once syntax is encoded as numbers, statements *about* syntax become statements *about* numbers. The system, which was originally only able to talk about counting, can now talk about its own theorems.

This embedding is the principle that gets reused everywhere:

- **Turing machines and the universal machine**: encoding programs as numbers, so a single program can simulate any other.
- **Kleene's recursion theorem**: every program has access to its own source code (a fixed-point combinator for recursive functions; cf. *The Y combinator* in this book).
- **Lawvere's fixed-point theorem**: in any sufficiently rich category, every endomorphism has a fixed point. A general category-theoretic version of the diagonal lemma.
- **Tarski's undefinability of truth**: there is no arithmetic formula that defines truth-in-arithmetic, by the same diagonal trick.
- **Halting problem**: undecidable by encoding programs as inputs.

All of these are versions of Gödel's coding trick. Each one needs an encoding (programs as data, formulas as numbers, sets as elements) and a diagonal-style application that constructs a self-referring object.

## What the trick costs

The encoding is *exponential*: the Gödel number of a 100-character string is roughly \\(p_{100}^{\text{max code}} \approx 541^{20}\\), an astronomical number. This is fine for theory; the encoding is just a mathematical device, not an efficient compression. Modern proof-assistant implementations use saner encodings (de Bruijn indices, structured terms), but for the metamathematical theorems, the exponential coding is enough.

For practical computation (compilers, interpreters, proof checkers), the principle survives but the encoding gets engineered for efficiency. The *fact* that programs can be encoded as data is what matters; the specific encoding is detail.

## The wonder

Before Gödel, the standard view was that arithmetic was a closed system: you have axioms, you derive theorems, you settle every statement of arithmetic eventually. Hilbert's program proposed to prove the consistency of mathematics itself by such derivations.

Gödel's coding trick revealed that arithmetic is *not* closed. By encoding sentences as numbers, he made arithmetic capable of talking about itself, and then he showed that this self-reference forces incompleteness: any consistent system containing arithmetic has true statements it cannot prove. The proof was the formalization of "this sentence is unprovable" — once you can express that sentence within arithmetic, the rest is logic.

The wonder is in the universality. The same coding trick — embed your syntax into your domain, then exploit the resulting self-reference — produces incompleteness, undecidability, fixed points, and Russell-style paradoxes throughout mathematics and computer science. It is one of the deepest patterns in formal reasoning. Whatever a system can talk about, it can usually talk about its own descriptions, and once it can do that, it can construct sentences that it cannot resolve.

## Where to go deeper

- Gödel, *Über formal unentscheidbare Sätze der Principia Mathematica und verwandter Systeme I*, 1931. The original.
- Smullyan, *Gödel's Incompleteness Theorems* (Oxford, 1992). Modern, accessible.
- Hofstadter, *Gödel, Escher, Bach* (1979). The popular introduction; long, brilliant, full of related material.
