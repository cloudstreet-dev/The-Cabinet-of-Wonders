# Univalent foundations

In standard mathematics, two structures are "the same" if they are isomorphic, but two definitions of the same structure are not, in any strict logical sense, equal. The integers built as equivalence classes of pairs of naturals, and the integers built as positive-or-zero-or-negative-naturals, are not literally the same set. They are isomorphic, and we treat them as the same in practice, but the formal language of set theory cannot express this directly.

Voevodsky's *univalence axiom* makes the informal practice formal. It says: in the right kind of foundation, *isomorphic structures are equal*. Not just equivalent — equal, as identities. Mathematics can then be done up to isomorphism by default, with no need for cumbersome translations. The price: the foundation has to be type theory with a particular structural property, not set theory.

This was Voevodsky's last major project before his death in 2017. The Univalent Foundations program is the most ambitious modern attempt to rebuild mathematical foundations on a basis better suited to how mathematics is actually practiced. It is the seed of homotopy type theory (HoTT), which is now part of the architecture of modern proof assistants like Coq, Lean (in its homotopy-flavored version), and Agda.

## The setup

In Martin-Löf type theory (the basis for Coq, Agda, Lean), every type \\(A\\) has an *identity type* \\(\text{Id}_A(x, y)\\), or \\(x =_A y\\) — the type of "proofs that \\(x\\) and \\(y\\) are equal as elements of \\(A\\)." For most types, this identity type is fairly simple.

For *types themselves* (as elements of a universe \\(\mathcal{U}\\)), the identity type \\(A =_\mathcal{U} B\\) is the type of "proofs that \\(A\\) and \\(B\\) are equal as types." Voevodsky's question: what is this identity type?

Naively, you might say: \\(A =_\mathcal{U} B\\) is "either \\(A\\) and \\(B\\) are syntactically the same, or they are not." But this is too restrictive — we want it to capture more than literal syntactic equality.

## The univalence axiom

Voevodsky's answer: \\((A =_\mathcal{U} B) \simeq (A \simeq B)\\).

The identity type between two types in the universe is *equivalent* to the type of equivalences between them. Two types are equal exactly when they are equivalent. (Here \\(A \simeq B\\) means there is a function \\(f : A \to B\\) and inverse \\(g : B \to A\\), with appropriate coherence conditions.)

So *isomorphic types are equal*. Provable.

This is the univalence axiom. In its presence, you can take any property \\(P(A)\\) about a type \\(A\\), and if \\(A \simeq B\\), then \\(P(A) \simeq P(B)\\). Properties transport along equivalences, automatically. This is what mathematicians have always done informally; univalence makes it a theorem.

## Why this matters

In set-theoretic foundations, "the integers" might mean any of several constructions. Each is a specific set, with specific element-objects. Properties of "the integers" are formally properties of specific sets. To transfer a theorem proved for one construction to another, you have to manually translate via the isomorphism — a tedious bookkeeping that mathematicians elide informally.

In univalent foundations, this is automatic. You define "the integers" up to equivalence; properties transport along that equivalence. Theorems proved for one realization apply to any equivalent one.

For mathematics in proof assistants, this is a huge ergonomic gain. You can choose a representation for ease of definition, prove your theorems, and apply them to whatever isomorphic representation you need — no manual translation.

## Homotopy type theory (HoTT)

Univalent foundations connects type theory to homotopy theory. The identity types form a structure called an *\(\infty\)-groupoid* — at each level, identifications between identifications between identifications form a structure that mathematicians studying topological spaces have known about for decades.

Specifically: types correspond to *spaces*. Identity types correspond to *path spaces*. Univalence is the statement that two spaces are equal iff they are homotopy equivalent. Functions between types correspond to continuous maps between spaces.

Under this dictionary, type theory becomes a *formal language for homotopy theory*, and homotopy-theoretic intuitions translate into type-theoretic constructions. Many concepts that are technical in classical homotopy theory (\\(n\\)-types, equivalences, fibrations, the Eilenberg-Steenrod axioms) become natural type constructions.

## What it enables: synthetic mathematics

In univalent foundations, you can do *synthetic* mathematics — mathematics where the basic objects are types with a structural property, rather than sets with construction-specific elements.

- **Synthetic homotopy theory**: prove theorems about spheres, loops, fundamental groups, in pure type theory, without ever defining "topological space" or "continuous function." The types and their identities give you the homotopy directly.
- **Synthetic algebraic topology**: similar for cohomology, K-theory, etc.
- **Cubical type theory**: a computational variant of HoTT where univalence is not just an axiom but a *computation rule*. Built into the proof assistant Cubical Agda. Functions on types automatically transport along equivalences.

The pitch is: instead of choosing arbitrary set-theoretic encodings of mathematical structures, work directly in a foundation where the structures are exactly what you defined them as, up to equivalence.

## What this changes for proof assistants

Modern proof assistants (Coq, Agda, Lean) are based on dependent type theory. Adding univalence (or working in a cubical extension) gives them the ergonomic benefits described above. For mathematics formalization, this is a real improvement.

The Liquid Tensor Experiment (Scholze and the Lean community, 2020-2022) formalized a cutting-edge piece of modern mathematics in Lean. The work involved many technical structural transports, and proof-assistant tooling around equivalence and univalence-style reasoning was actively developed during the project.

Lean 4's mathlib, Coq's HoTT library, and Cubical Agda's library are all developing infrastructure for univalent-foundations-style mathematics. The community is large and growing.

## Where it does not (yet) help

Univalent foundations is a research project, not yet a finished foundation:

- The univalence axiom is *consistent* with type theory (Voevodsky's simplicial-set model), but its full computational interpretation is the subject of cubical-type-theory research. In Agda's standard mode and in Lean (as of 2025), univalence is an axiom, not a computation.
- Many classical theorems require classical logic (excluded middle, axiom of choice) which interacts subtly with univalence. The right combinations are still being worked out.
- Higher categorical structures beyond \\((\infty, 1)\\)-categories require even more elaborate type-theoretic frameworks.

## The wonder

Mathematicians have always done their work *up to isomorphism*. "The integers" is whatever satisfies the Peano axioms, regardless of construction. "The cyclic group of order 7" is determined up to isomorphism by its presentation. The classical foundation, set theory, fights this by making the choice of encoding matter at the formal level, even when it does not matter mathematically.

Univalent foundations is a foundation that *does not fight*. It identifies isomorphic structures as equal at the formal level, matching the informal practice. Mathematicians can finally formalize their work in a system that respects how mathematics actually thinks.

The wonder is in the fit. Voevodsky was a working algebraic geometer who became dissatisfied with how cumbersome formalizing his subject would be in classical foundations. He invented (or co-invented, with several collaborators) a foundation that fits the working geometer's practice. The fit is so natural that, in retrospect, it seems strange we ever did it any other way.

The downside: the foundation is more complex. Set theory is two pages of axioms; univalent foundations is type theory plus the univalence axiom plus higher inductive types plus the technical machinery to make all this computational. The complexity is the price of the structural fit.

## Where to go deeper

- *Homotopy Type Theory: Univalent Foundations of Mathematics* (the HoTT Book), 2013. The collective monograph from the IAS year on the subject. Free online.
- Voevodsky's notes and lectures, available from the Univalent Foundations Project website.
