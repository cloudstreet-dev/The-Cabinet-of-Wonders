# Mass point geometry

A geometry problem about ratios of segments inside a triangle, the kind that takes half a page of similar-triangle arguments to solve, can be reduced to balancing weights at the vertices and reading off the answer in three lines. The technique is taught to high-schoolers preparing for the Olympiad, and it works because Archimedes' law of the lever is, secretly, a theorem about ratios in the plane.

## The setup

Take a triangle ABC with cevians (lines from a vertex to the opposite side) that intersect inside it. The standard problem: given that points D, E divide certain sides in known ratios, find the ratio in which the cevians cut each other.

The classical solution uses Menelaus, Ceva, similar triangles, or barycentric coordinates. All of them work and all of them are tedious. Mass point geometry produces the same answer with arithmetic.

## The mechanism

Assign a positive number — call it a *mass* — to each vertex. The point with mass \\(m\\) at position \\(P\\) is denoted \\(m \cdot P\\). Two such mass points combine into one:

\\[ m_A \cdot A + m_B \cdot B = (m_A + m_B) \cdot G \\]

where \\(G\\) is the unique point on segment \\(AB\\) such that \\(\frac{AG}{GB} = \frac{m_B}{m_A}\\). Heavy side wins, which is to say the balance point lies closer to the heavy mass — exactly the law of the lever.

That is the only rule, applied repeatedly.

## A worked example

Triangle ABC. Point D on BC with \\(BD:DC = 2:3\\). Point E on AC with \\(AE:EC = 4:1\\). Cevians AD and BE meet at point P. Find \\(AP:PD\\) and \\(BP:PE\\).

The solution begins by choosing masses so that the cevian endpoints balance:

D is on BC with \\(BD:DC = 2:3\\). For D to be the balance point of B and C, we need \\(m_B : m_C = 3 : 2\\) (heavy mass closer). Let \\(m_B = 3\\), \\(m_C = 2\\). Then \\(D\\) has mass \\(m_B + m_C = 5\\).

E is on AC with \\(AE:EC = 4:1\\). We need \\(m_A : m_C = 1 : 4\\). C already has mass 2, so we need \\(m_A = 2/4 = 1/2\\). Multiply everything by 2 to clear the fraction: \\(m_A = 1\\), \\(m_C = 4\\). But we already set \\(m_C = 2\\). Multiply the first assignment by 2 to keep C consistent: \\(m_B = 6\\), \\(m_C = 4\\), and \\(m_A = 1\\). Then \\(D\\) has mass \\(m_B + m_C = 10\\) and \\(E\\) has mass \\(m_A + m_C = 5\\).

P is the intersection of AD and BE. On segment AD, P is the balance point of A (mass 1) and D (mass 10), so

\\[ AP : PD = 10 : 1 \\]

On segment BE, P is the balance point of B (mass 6) and E (mass 5), so

\\[ BP : PE = 5 : 6 \\]

Done.

```
                   A (1)
                  /\
                 /  \
                /    E (5)  AE:EC = 4:1
               /     \
              /   P   \
             /         \
            /           \
           /             \
          B-------D-------C
          (6)    (10)    (4)
                BD:DC = 2:3
```

The whole computation is: pick masses so the given ratios are satisfied, multiply through to make the masses consistent at shared vertices, then read off the ratios at the intersection.

## Why it works

The vector form makes it transparent. Let \\(A, B, C\\) be position vectors. The point dividing \\(BC\\) in ratio \\(BD:DC = m_C : m_B\\) is

\\[ D = \frac{m_B B + m_C C}{m_B + m_C} \\]

This is exactly the center of mass of the two-particle system \\(\{(m_B, B), (m_C, C)\}\\). Composition of mass points is composition of subsystems by the standard center-of-mass formula:

\\[ \text{center of mass}\big(\text{system}_1 \cup \text{system}_2\big) = \frac{M_1 \bar{x}_1 + M_2 \bar{x}_2}{M_1 + M_2} \\]

where \\(M_i\\) and \\(\bar{x}_i\\) are the total mass and center of mass of subsystem \\(i\\). The whole machinery is just this one formula, applied associatively.

The intersection of two cevians is a point, and that point is the center of mass of the entire three-vertex system from two different decompositions:

\\[ \underbrace{m_A A + (m_B B + m_C C)}_{\text{decompose along AD}} = \underbrace{m_B B + (m_A A + m_C C)}_{\text{decompose along BE}} = m_A A + m_B B + m_C C \\]

The ratios at the cevian-intersection point fall out of which two masses sit on which side of the balance.

## Why the framing is the magic

You could solve the same problem with vectors directly. The vector solution would require setting up coordinates, expressing each cevian parametrically, solving a 2×2 system. Mass point geometry says: do not bother with coordinates. The ratio you want is just a ratio of masses, and the masses are determined by the ratios you were given. The whole problem collapses into bookkeeping.

The reframing — recognizing that segment ratios obey the same algebra as the lever — is what makes the technique feel like a sleight of hand. Once you see it, you cannot un-see it. Geometry problems start to look like they are presenting their own answers.

## Limits and extensions

Mass points handle cevians in a triangle, and any ratios derived from them. Three cevians concurrent at a single point are easy. Four-line problems and configurations involving the *outside* of segments need signed masses (negative weights for points on the extension of a segment), which the technique extends to.

For more general projective geometry, the right framework is barycentric coordinates: every point in the plane of the triangle has unique coordinates \\((\alpha : \beta : \gamma)\\), with \\(\alpha + \beta + \gamma\\) optionally normalized, and the same balance algebra applies. Mass points are barycentric coordinates with the projection to the triangle's interior; signed mass points are barycentric coordinates without restriction.

## The wonder

Archimedes proved that levers balance when the products of weight and distance match. Two thousand years later it turned out that the same identity is the entire content of "if a cevian crosses another cevian, where do they meet?" — a question Archimedes never asked, in a context (Olympiad geometry) that did not exist yet. The mathematics did not know it was supposed to be about levers. Levers and segment ratios share an algebra because both are weighted averages of points in space, and weighted averages are weighted averages.

## Where to go deeper

- Tom Rike, *A Beautiful Application of Archimedes' Lever* (Berkeley Math Circle notes). The cleanest short introduction.
- Coxeter, *Introduction to Geometry*, Chapter 13 on barycentric coordinates. The full theory of which mass points are a special case.
