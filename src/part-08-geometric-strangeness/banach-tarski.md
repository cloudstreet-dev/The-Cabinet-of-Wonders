# Banach–Tarski

You can take a solid sphere, cut it into five pieces, move and rotate them rigidly without stretching or compressing, and reassemble them into *two* solid spheres, each the same size as the original. No mass is created. No mass is lost. The pieces are exactly the original sphere; the result is exactly two copies of it. The total volume doubles, in clear violation of every intuition about how rigid motions work.

This is not a trick. It is a theorem of Banach and Tarski (1924), provable from the axioms of standard set theory, and impossible only because the pieces are not measurable in the ordinary sense. They have no volume. They cannot have one. Once you accept that, the doubling makes sense.

The result is a wonder by negation: an existence proof that something we are sure is impossible is, in the formal axiomatic sense, possible. It exposes how much our physical intuition leans on Lebesgue-measurability of pieces.

## The setup

A *paradoxical decomposition* of a set \\(X\\) under a group \\(G\\): a partition \\(X = A_1 \cup A_2 \cup \dots \cup A_n\\) such that, for some elements \\(g_i \in G\\), \\(g_i(A_i)\\) for \\(i \in I_1\\) tile \\(X\\) exactly, *and* \\(g_i(A_i)\\) for \\(i \in I_2\\) (the complementary index set) also tile \\(X\\) exactly. So a single decomposition of \\(X\\) yields, by group transformation, two copies of \\(X\\).

Banach-Tarski: the unit sphere \\(S^2\\) (or the closed unit ball \\(B^3\\)) admits a paradoxical decomposition under the group of rotations of \\(\mathbb{R}^3\\) (or rigid motions for \\(B^3\\)).

The number of pieces can be made as small as 5 (Robinson 1947). Three of them give one sphere; the other two give the second; well, with one center point as a special case, the actual count is more like five if you are careful about boundaries.

## Why it works: paradoxical groups

The proof rests on a property of the rotation group \\(SO(3)\\): it contains a *free subgroup of rank 2*. Specifically, two rotations \\(\rho, \sigma\\) by appropriate angles around appropriate axes generate a free group \\(F_2\\) — every element of the group is a unique non-trivial reduced word in \\(\rho, \rho^{-1}, \sigma, \sigma^{-1}\\). No nontrivial relations between them.

The free group \\(F_2\\) has its own paradoxical decomposition. Partition \\(F_2\\) into:

- \\(W(\rho)\\): words starting with \\(\rho\\).
- \\(W(\rho^{-1})\\): words starting with \\(\rho^{-1}\\).
- \\(W(\sigma)\\): words starting with \\(\sigma\\).
- \\(W(\sigma^{-1})\\): words starting with \\(\sigma^{-1}\\).
- \\(\{e\}\\): the identity.

Then \\(\rho \cdot W(\rho^{-1}) \cup W(\rho) = F_2 \setminus \{e\}\\) — applying \\(\rho\\) to all words starting with \\(\rho^{-1}\\) cancels the \\(\rho^{-1}\\), giving every word *not* starting with \\(\rho\\); together with \\(W(\rho)\\) you get every non-identity word. Similarly \\(\sigma \cdot W(\sigma^{-1}) \cup W(\sigma) = F_2 \setminus \{e\}\\). So two of the four parts, each shifted by one rotation, give two copies of \\(F_2\\) (modulo the identity, handled separately). The free group decomposes into pieces that, after rotating, double.

Now propagate this to the sphere. \\(F_2\\) acts on \\(S^2\\) by rotations. The sphere decomposes into orbits under \\(F_2\\). Pick a representative point from each orbit (this is where the *Axiom of Choice* enters). The orbit of any chosen point under \\(F_2\\) inherits the paradoxical decomposition of \\(F_2\\). Stitch the orbit-wise decompositions together into a sphere-wide one.

A measure-zero set of fixed points (the rotational axes) needs to be handled with extra care — they are dealt with by absorbing them into the "main" decomposition. The result is the paradoxical decomposition of \\(S^2\\), then \\(B^3\\).

## Why this is not a contradiction

The pieces are *not Lebesgue measurable*. They cannot be assigned a sensible volume. Volume is countably additive over disjoint countable unions; a paradoxical decomposition would require \\(\text{vol}(B^3) = \text{vol}(B^3) + \text{vol}(B^3) = 2 \cdot \text{vol}(B^3)\\), giving \\(\text{vol}(B^3) = 0\\) or \\(\infty\\), which is wrong. So at least one of the pieces must be non-measurable. The Axiom of Choice constructs a non-measurable selection of orbit representatives, and from there the entire decomposition inherits non-measurability.

If you reject the Axiom of Choice (or replace it with weaker axioms like Dependent Choice plus "every set of reals is Lebesgue measurable"), Banach-Tarski fails. Solovay (1970) showed there are models of ZF + DC where every set of reals is measurable, and in such models, Banach-Tarski is false.

## Why \\(\mathbb{R}^2\\) is different

Banach-Tarski does not work in dimension 2. The plane has a *finitely additive measure invariant under rigid motions* extending Lebesgue measure to all subsets. (Banach himself proved this.) So no paradoxical decomposition of the disk is possible in two dimensions.

The reason: the group of rigid motions of the plane is *amenable* (it has an invariant mean), while the group of rigid motions of three-dimensional space is *not* amenable — it contains a free non-abelian subgroup. Amenability of the group blocks paradoxical decomposition; non-amenability is necessary. This is the Tarski theorem (1929): a group acts paradoxically on a set with finitely many pieces iff the group is non-amenable.

So the line and plane have a "no paradoxical decomposition" theorem, while three-space and higher do not. The break in behavior at dimension 3 is because that is where the rotation group becomes large enough to contain a free subgroup.

## What "rigid motion" means

Important detail: the pieces are moved by rigid motions — translations and rotations, no stretching, no compression, no scaling. The volumes of the pieces (if they had any) would be preserved by these motions. The fact that this set of allowed operations can double a sphere is the surprise. Rigid motions in \\(\mathbb{R}^3\\) preserve everything that has a measure; they fail to preserve measure only because the pieces lack one to begin with.

## Five pieces, no fewer

Robinson (1947) proved that 5 is the minimum number of pieces. Wilson (2005) gave a particularly clean version with 5 pieces by direct construction, using *abused* word-shifts in the free group.

The construction is far from explicit. The pieces are non-measurable, hence not constructively describable; the proof exhibits them only through the Axiom of Choice. You cannot draw a Banach-Tarski decomposition. You can only prove its existence.

## Why this matters

Mostly it does not, in the engineering sense — there is no "Banach-Tarski algorithm" to deploy. Its importance is foundational:

**It demonstrates the necessity of measure theory.** Real-world geometric reasoning works because we restrict to measurable sets. Without that restriction, intuition collapses.

**It exemplifies the role of the Axiom of Choice.** Choice constructs sets that "should not exist" by ordinary intuitions. Banach-Tarski is the strongest, most striking example.

**It shapes the theory of amenable groups.** Tarski's theorem characterizing paradoxical actions in terms of group amenability is central to large parts of geometric group theory.

**It clarifies what physical intuition assumes.** Volume preservation under rigid motion is so intuitive that it feels like a logical necessity. Banach-Tarski says: no, it is a consequence of measurability, and there are sets to which it does not apply. Physics protects us from these because physical objects are made of (countably many, regular) atoms.

## The wonder

You can decompose a ball into five pieces and reassemble them into two balls.

Not "stretch and compress." Not "infinitely subdivide." Not "smear into points." Five pieces, rigid motions, two balls. The pieces are weird — non-measurable, dust-like, intricate beyond drawing — but they are mathematical sets, and the construction is a theorem in standard set theory.

The wonder is that volume preservation under rigid motion is *not* a logical inevitability. It depends on measurability. Drop measurability — which the Axiom of Choice forces you to consider — and you get sphere doubling. The mathematical universe is larger and stranger than the physical universe constrains us to imagine.

## Where to go deeper

- Wagon, *The Banach-Tarski Paradox*, Cambridge 1985. The book on this. Read everything.
- Tao, *The Banach-Tarski Paradox* (blog post). Modern, clean exposition.
