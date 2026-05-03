# Forcing

You have a model of set theory \\(M\\). You wish it satisfied a particular statement — say, "there are more real numbers than \\(\aleph_1\\)" (the negation of the Continuum Hypothesis). Forcing is a technique that lets you *extend* \\(M\\) to a larger model \\(M[G]\\) in which the desired statement holds, while preserving all the axioms of ZFC. The new "elements" of \\(M[G]\\) are not chosen arbitrarily; they are guaranteed to exist by a careful technical construction in \\(M\\) itself.

Cohen invented forcing in 1963 to prove the independence of the Continuum Hypothesis from ZFC. He won the Fields Medal for it. The technique remains the most powerful tool in set theory for proving statements unprovable.

It is dense, technical, and unwieldy. The wonder is in the result: you can construct a universe of mathematics where, say, there are exactly \\(\aleph_2\\) reals (or \\(\aleph_{17}\\), or any cardinality consistent with ZFC), and another where there are \\(\aleph_1\\), and ZFC cannot tell them apart.

## The setup

Cohen's question: is the Continuum Hypothesis (CH) — "every infinite subset of \\(\mathbb{R}\\) has either the size of \\(\mathbb{N}\\) or the size of \\(\mathbb{R}\\), with no intermediate cardinality" — provable from ZFC?

Gödel had shown (1940) that CH is *consistent* with ZFC: there is a model of ZFC (the constructible universe \\(L\\)) in which CH holds. So ZFC cannot disprove CH.

Cohen needed to show the *converse*: there is a model of ZFC in which CH fails. So ZFC cannot prove CH either. Then CH is independent of ZFC.

The challenge: how do you build a model of ZFC where CH is false? You cannot "just" add reals to an existing model — naively adding things might break some other axiom. You need a controlled extension.

## The idea

Start with a *countable transitive model* \\(M\\) of ZFC. (Such models exist by Löwenheim-Skolem.) The model contains some sets, including some real numbers — exactly \\(\aleph_1^M\\) of them (where \\(\aleph_1^M\\) is what \\(M\\) thinks is the first uncountable cardinal).

We want to add more real numbers, enough to make CH false. We will add a *generic* set \\(G\\) — a specific subset of some pre-defined poset \\(P \in M\\) — such that the new universe \\(M[G]\\) contains many new reals.

The key constraints:

- \\(M[G]\\) must satisfy all axioms of ZFC.
- \\(M[G]\\) must contain enough new reals to falsify CH.
- \\(M[G]\\) must agree with \\(M\\) on the old sets and on cardinalities (or, the cardinalities can be controlled by choosing the right poset).

## The poset and forcing conditions

Pick a partially-ordered set \\(P \in M\\). Elements of \\(P\\) are *forcing conditions*. For Cohen forcing on adding \\(\aleph_2\\)-many reals:

\\[ P = \{ p : p \text{ is a finite partial function from } \aleph_2 \times \mathbb{N} \to \{0, 1\} \} \\]

Each \\(p\\) is a finite "approximation" to a function \\(F: \aleph_2 \times \mathbb{N} \to \{0, 1\}\\), specifying \\(F\\)'s value at finitely many points. The order: \\(q \leq p\\) means \\(q\\) extends \\(p\\) — \\(q\\) decides everything \\(p\\) decides plus more.

A *generic filter* \\(G \subseteq P\\) is a filter (chain-like subset) that meets every dense set definable in \\(M\\). Such a filter exists because \\(M\\) is countable: there are countably many dense sets in \\(M\\), and you can construct \\(G\\) to meet all of them.

The union \\(\bigcup G\\) is a function \\(F : \aleph_2 \times \mathbb{N} \to \{0, 1\}\\). For each \\(\alpha < \aleph_2\\), \\(F(\alpha, \cdot)\\) is a binary sequence — encoding a real number. So \\(G\\) gives \\(\aleph_2\\)-many distinct new reals.

These reals are not in \\(M\\) — \\(G\\) is generic, so it is not in \\(M\\), and the new reals it codes are not in \\(M\\). They get added to the larger model \\(M[G]\\).

## The forcing relation

The technical work is showing \\(M[G] \models \text{ZFC}\\), and that \\(M[G]\\) sees \\(\aleph_2\\) new reals (so CH fails).

The trick is the *forcing relation* \\(p \Vdash \varphi\\) — "the condition \\(p\\) forces the formula \\(\varphi\\)." The relation is definable in \\(M\\), without any reference to \\(G\\). Roughly, \\(p \Vdash \varphi\\) means: every generic filter containing \\(p\\) satisfies \\(\varphi\\) in the resulting extension.

Cohen showed:

- The forcing relation is definable in \\(M\\).
- For every formula \\(\varphi\\), either some \\(p \in P\\) forces \\(\varphi\\) or some \\(p\\) forces \\(\neg \varphi\\) (and one of them is in any given generic filter).
- The forcing extension \\(M[G]\\) satisfies the formulas forced by elements of \\(G\\).

Working through this, you check each ZFC axiom: does the forcing extension satisfy it? For Cohen's poset, all ZFC axioms are preserved, and the cardinal \\(\aleph_2\\) is preserved (\\(P\\) is "countably-closed" or has a related property; cardinality counting in \\(M\\) and \\(M[G]\\) match).

The result: \\(M[G] \models \text{ZFC}\\), and \\(M[G]\\) has \\(\aleph_2\\)-many reals, so CH is false in \\(M[G]\\). QED.

## Variants

By choosing different posets, you can engineer extensions with various properties:

- **Cohen forcing**: adds many "generic" reals. Falsifies CH.
- **Random forcing**: adds reals from a Lebesgue-measure-zero side. Used in measure-theoretic independence results.
- **Sacks forcing, Laver forcing, Mathias forcing, Prikry forcing**: each adds reals with a specific structural property. Used for various independence results.
- **Iterated forcing** (Solovay-Tennenbaum): iterate forcing transfinitely many times to add many generic objects in a controlled way. Used to prove the Suslin Hypothesis is independent.
- **Class forcing**: forcing where the poset is a proper class, not a set. More technical but allows even larger extensions.

There are entire books on different forcing notions and what they can establish.

## What forcing has accomplished

Beyond CH:

- **Suslin Hypothesis**: every Dedekind-complete totally-ordered set without endpoints, in which every collection of disjoint intervals is at most countable, is order-isomorphic to \\(\mathbb{R}\\). Independent of ZFC (forcing).
- **Whitehead's problem**: every \\(\aleph_1\\)-free abelian group is free. Independent of ZFC.
- **Borel determinacy** is provable in ZFC, but its strength requires significant set-theoretic apparatus. Higher determinacy axioms are independent.
- **Existence of large cardinals**: many statements about large cardinals are independent of ZFC, with relative-consistency results obtained by forcing extensions.

The general framework: take a problem from analysis, topology, algebra, combinatorics; ask whether ZFC settles it; if not, use forcing to construct two models, one where it holds and one where it does not.

## Why this is wonder

Set theory was, for most of its history, considered the *foundation* of mathematics — the bedrock on which everything else stood. Forcing showed that the bedrock has *cracks*. Many natural mathematical questions cannot be settled by ZFC.

Worse: forcing is *constructive*. It does not just say "CH is independent"; it gives you, by name, a model where CH holds and a model where CH fails. You can do mathematics in either, internally consistently. The "real" answer is undefined.

This led to a 60-year debate in the foundations community: is mathematics a single object whose truth we are uncovering, or a multiverse of consistent universes? Cohen's own view leaned toward the latter. Gödel's leaned toward the former, with the conviction that "natural" axioms beyond ZFC will eventually settle the open questions.

The technical wonder is that the construction works: you can extend a model of set theory to a larger model satisfying all the same axioms plus a chosen new statement, and the technique is *uniform* — choose any reasonable poset and the construction goes through.

## Where to go deeper

- Cohen, *Set Theory and the Continuum Hypothesis*, 1966. The book.
- Kunen, *Set Theory: An Introduction to Independence Proofs*, North-Holland 1980. The standard reference for forcing.
- Chow, *A Beginner's Guide to Forcing*, arXiv 2007. Genuinely accessible.
