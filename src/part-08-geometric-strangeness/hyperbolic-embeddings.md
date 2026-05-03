# Hyperbolic embeddings

A tree with \\(2^{30}\\) leaves cannot be embedded in Euclidean space without distortion: the leaves want to be uniformly spread, but Euclidean space's volume grows polynomially in the radius (\\(O(r^d)\\) in \\(d\\) dimensions), while a tree's volume grows exponentially in depth. Squeeze a billion-leaf tree into a Euclidean ball, and most of the leaves bunch up; geodesic distances are wildly different from tree distances.

In *hyperbolic* space, volume grows *exponentially* with radius. The geometry matches the tree's growth rate. A two-dimensional hyperbolic disk can isometrically embed any tree with bounded distortion. Three-dimensional hyperbolic space embeds graphs with hierarchical structure beautifully — better, often, than thousand-dimensional Euclidean spaces.

This was understood theoretically since Lobachevsky and Bolyai (1820s-30s). It was applied to embedding hierarchical graphs (the Internet, social networks, taxonomies) only in the late 2000s, and it has become a recurring tool in machine learning's representation problems for graph-structured data.

## Hyperbolic geometry, briefly

Hyperbolic space is a Riemannian manifold of constant negative curvature. The Poincaré disk model represents 2D hyperbolic space as the open unit disk in \\(\mathbb{R}^2\\); the hyperbolic metric is

\\[ d s^2 = \frac{4 (d x^2 + d y^2)}{(1 - x^2 - y^2)^2} \\]

Distances *grow* as you approach the boundary. A "small" Euclidean step near the boundary corresponds to a long hyperbolic distance. The boundary is "infinitely far" in hyperbolic distance, but it sits inside a unit disk in the embedded representation.

```
   Poincaré disk
   ___________
  /     .    \\         . . . . . . .
 /  .       .  \\        .         .
|  .   ___   .  |   <-- 2 lines that look      
|  .  / o  \\ . |       curved here are
|  . |  o   | . |       straight lines in
|  .  \\ o /  . |       hyperbolic geometry
 \\  .       .  /
  \\____.____  /        boundary at infinity
```

In this model, hyperbolic geodesics (straight lines) are arcs of circles that meet the boundary perpendicularly (or diameters). Two geodesics that look like they should meet ("parallel lines") instead diverge exponentially.

The volume of a hyperbolic ball of radius \\(r\\) grows as \\(\sinh(r) \sim e^r / 2\\) for large \\(r\\): exponentially in the radius. So a hyperbolic 2D disk has *exponentially* more "room" at large radii than a Euclidean 2D disk does — the room scales like the perimeter, which grows exponentially.

## Trees embed naturally

A balanced binary tree of depth \\(d\\) has \\(2^d\\) leaves. The natural embedding into hyperbolic 2D: place the root at the center; place each level at a hyperbolic distance \\(\delta\\) further out; spread the children of each node uniformly in angle. The number of points placeable at radius \\(r\\) in hyperbolic 2D, with separation \\(\delta\\), is exponential in \\(r\\). So all \\(2^d\\) leaves fit at hyperbolic radius \\(O(d)\\), with hyperbolic distances between sibling leaves bounded.

In contrast, Euclidean 2D: \\(2^d\\) leaves need to be placed at radius \\(O(\sqrt{2^d})\\) — exponentially far from the root — to avoid bunching. So tree distances and Euclidean distances diverge wildly.

Tree embeddings into Euclidean space have *distortion* lower-bounded by \\(\Omega(\sqrt{d})\\) for trees of depth \\(d\\). Hyperbolic 2D embeds with constant distortion — better than Euclidean of any finite dimension.

## Real-world graphs and the Internet

Empirically, the Internet's autonomous-system topology, the World Wide Web's link structure, social networks, and many biological networks have graph distances that look more like tree distances than like Euclidean distances. They have small diameter, exponential expansion, hyperbolic-like curvature.

Boguñá, Papadopoulos, Krioukov (2010) embedded the Internet into 2D hyperbolic space. Result: each AS corresponds to a point in the hyperbolic disk; the "ground-truth" hyperbolic distance correlates strongly with hop-count distance. Greedy routing — at each step, forward to the neighbor closest to the destination in hyperbolic coordinates — succeeds on the Internet topology with very high probability and short paths. This is the basis of *hyperbolic geographic routing* schemes.

## Machine-learning applications

Embedding graph-structured data (taxonomies, ontologies, knowledge graphs, social networks) into vector spaces, for use in downstream ML tasks, was traditionally done in Euclidean space (word2vec, node2vec, GNN encoders). Nickel and Kiela (2017) showed *hyperbolic embeddings* often outperform Euclidean embeddings of much higher dimension for hierarchical data.

The intuition: the hierarchy's inherent tree-like structure matches hyperbolic geometry. A 5-dimensional hyperbolic embedding of WordNet (a hierarchical lexical database) outperforms 200-dimensional Euclidean embeddings on link-prediction tasks. The geometry is doing the work that dimensions had to do in Euclidean space.

Modern hyperbolic neural networks (Ganea et al., Chami et al.) build on this: hyperbolic versions of multilayer perceptrons, attention, graph convolutions. Useful in domains where data has tree-like or fractal structure.

## Hyperbolic embeddings of social networks

Krioukov-Papadopoulos-Kitsak-Vahdat-Boguñá (2010) embedded social networks into 2D hyperbolic. Friendship probability looks like a function of hyperbolic distance, with sharp threshold. Geographic and demographic data correlates with hyperbolic-coordinate cluster structure. Predictive of future link formation.

This builds the case that real social networks live in low-dimensional hyperbolic space, even though they are usually represented in Euclidean.

## Why high curvature

Curvature \\(K\\) of a Riemannian manifold determines volume growth. \\(K = 0\\) (flat space): polynomial growth. \\(K > 0\\) (sphere): bounded growth. \\(K < 0\\) (hyperbolic): exponential growth.

To embed an exponentially-growing structure (tree, scale-free graph, fractal) without distortion, you want a metric with exponential volume growth — so you want negative curvature. Hyperbolic space provides it natively.

You can also use spaces of mixed curvature (product manifolds: spherical × hyperbolic × Euclidean) for graphs with mixed structure. The general theory of *non-Euclidean ML* is an active area.

## What it costs

Hyperbolic computation has subtleties:

- Numerical precision near the boundary of the Poincaré disk degrades fast (denominators approach zero).
- Optimization on hyperbolic manifolds requires Riemannian gradient descent.
- Standard ML primitives (linear layers, attention) need hyperbolic versions, often involving Möbius arithmetic.

The trade-off pays off when the data has hierarchical structure. For data without obvious hierarchy, Euclidean is fine.

## The wonder

A tree with a billion leaves does not fit in any Euclidean space without serious distortion — the structure's volume growth (exponential) and Euclidean volume growth (polynomial) are fundamentally mismatched. But there is a geometry — hyperbolic geometry, formalized in the 1820s as a curiosity of non-Euclidean axioms — whose volume growth matches the tree exactly. Trees fit naturally in hyperbolic 2D. Real-world graphs with hierarchical structure fit naturally too.

The wonder is not just that hyperbolic geometry exists. It is that *real data* — the Internet, social networks, taxonomies — has the same exponential growth rate, suggesting an underlying hyperbolic-like generative process. The geometry that two 19th-century mathematicians invented as a logical exercise turns out to be the natural habitat for the kinds of large hierarchical structures that 21st-century engineers and biologists encounter.

The dimensionality argument lands harder when you compare directly: 5-D hyperbolic outperforms 200-D Euclidean. The geometry is, in some quantifiable sense, more efficient at holding hierarchical relationships than Euclidean space at any finite dimension.

## Where to go deeper

- Bonahon, *Low-Dimensional Geometry*, AMS 2009. Modern introduction to hyperbolic geometry.
- Nickel and Kiela, *Poincaré Embeddings for Learning Hierarchical Representations*, NeurIPS 2017. The ML breakthrough.
