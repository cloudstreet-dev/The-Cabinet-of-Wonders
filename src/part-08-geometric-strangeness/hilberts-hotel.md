# Hilbert's hotel

A hotel with infinitely many rooms, all occupied, can accommodate one more guest by moving everyone up one room. It can accommodate a busload of one million more by moving each existing guest up by a million. It can accommodate a *countably infinite* busload of new guests by moving each existing guest at room \\(n\\) to room \\(2n\\), freeing all the odd-numbered rooms. And it can accommodate a *countably infinite collection of countably infinite busloads* by a slightly cleverer rearrangement.

Hilbert used the metaphor in lectures around 1924 to make set-theoretic cardinality concrete. It still is: the strangeness of infinity, distilled into a setting that anyone can picture, with consequences that do not match physical intuition at all.

## Adding one guest

Hotel has rooms 1, 2, 3, ..., all full. A new guest arrives. The manager makes an announcement: "Every guest, please move from your current room to the next-numbered room." Guest in 1 → 2, guest in 2 → 3, etc. Now room 1 is empty. The new guest takes it.

There is no room "at the end" left empty by this shift, because there is no end. The shift just moves everyone forward by one, and the bijection \\(n \mapsto n + 1\\) maps \\(\mathbb{N}\\) onto \\(\mathbb{N} \setminus \{1\}\\). One slot is freed at the start; no slot is opened up "at infinity."

## Adding a countable infinity of guests

A bus arrives with a countable infinity of new guests \\(g_1, g_2, g_3, \dots\\). The manager says: "Every current guest, please move from room \\(n\\) to room \\(2n\\)." Now odd rooms are all empty. The new bus's guest \\(g_k\\) goes to room \\(2k - 1\\).

The bijection \\(n \mapsto 2n\\) maps \\(\mathbb{N}\\) onto the even naturals; the original infinitely many guests are still housed, but now in only the even rooms. The odd rooms hold the new infinite stream.

## Countable infinity of buses, each with countable infinity of guests

A motorcade arrives: a countable sequence of buses \\(B_1, B_2, B_3, \dots\\), each carrying a countable infinity of guests \\(g_{1, k}, g_{2, k}, \dots\\). Total: \\(\aleph_0 \times \aleph_0 = \aleph_0\\) new guests.

One manager strategy: enumerate the new guests by Cantor's diagonal pairing function. Guest \\(g_{i,j}\\) is the \\(\binom{i+j}{2} + i\\)-th in the enumeration (or equivalent). After the existing guests move to the even rooms, assign new guests to the odd rooms in this enumeration order.

Or, more elegantly: assign each guest \\(g_{i,j}\\) (the \\(j\\)-th passenger of the \\(i\\)-th bus) to room \\(2^i \cdot 3^j\\). Existing guests move from room \\(n\\) to room \\(5^n\\). All assignments are unique (by uniqueness of prime factorization), so no two people share a room.

This still wastes most rooms (those whose factorizations contain primes other than 2, 3, 5), but that is fine — there are infinitely many to spare.

## What's actually happening

Hilbert's hotel illustrates that infinite sets can be in bijection with proper subsets of themselves. Galileo noticed this in 1638 ("more squares than non-squares?... but every natural number is a square root, so there are equally many"). Cantor formalized it. A set is *Dedekind-infinite* iff it is in bijection with a proper subset.

For finite sets, you cannot fit \\(n + 1\\) things into \\(n\\) boxes (pigeonhole). For \\(\mathbb{N}\\), you can fit \\(\mathbb{N} + 1\\) things into \\(\mathbb{N}\\) boxes — because \\(\mathbb{N}\\) and \\(\mathbb{N} + 1\\) have the same cardinality, even though one of them looks "bigger" by adding an element.

The pigeonhole principle, fundamental to combinatorics, depends on finiteness. Drop finiteness; you drop pigeonhole; you get Hilbert's hotel.

## The harder question: more buses than the hotel can hold

What if the motorcade has \\(\aleph_1\\) buses, each with \\(\aleph_0\\) passengers (or, replace this with an uncountable bus)? Cardinality \\(\aleph_1\\) total. Now the hotel cannot accommodate them: \\(|\mathbb{N}| = \aleph_0 < \aleph_1\\). A bijection cannot be set up.

Cantor's diagonal argument: there is no injection from \\([0, 1]\\) (which has cardinality \\(2^{\aleph_0} = \aleph_1\\) under CH) into \\(\mathbb{N}\\). So an uncountable bus could not be checked in.

The hotel handles countable infinities of countable additions trivially — anything that can be enumerated, the manager has a plan for. Uncountable additions break the hotel because no enumeration exists.

## The wonder

The disjunction between physical and set-theoretic intuition is the whole point. In a real hotel, "every room is full and a new guest arrives" is a contradiction — the hotel is a closed system, you cannot get more from it than was put in. In Hilbert's hotel, "every room is full and a new guest arrives" is a question that has an answer, and the answer is "yes, with a small reshuffle."

The reason real hotels do not behave this way is finiteness. The reason set-theoretic infinities do behave this way is that they are not finite. The behavior is not paradoxical — it is *required* by the definitions. The wonder is that you can communicate this to a person of any background by describing the hotel and watching them realize that they had been assuming finiteness all along.

The deeper wonder, perhaps: that there are *different* infinities, and the hotel can accommodate countable additions but not uncountable ones. The hotel illustrates one tier of the infinite cardinal hierarchy, and Cantor's theorem (every set has strictly more subsets than elements) shows there is a hierarchy reaching higher than \\(\aleph_0\\) without limit.

## Where to go deeper

- Hilbert's lectures on the infinite, 1925 (published in *On the Infinite*). The original.
- Smullyan, *Satan, Cantor, and Infinity*. Light, witty, full of related infinity-puzzles.
- Devlin, *The Joy of Sets* (2nd ed.). Modern set theory textbook with cardinality and ordinals.
