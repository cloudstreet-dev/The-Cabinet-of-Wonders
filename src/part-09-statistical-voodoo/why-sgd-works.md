# Why SGD works at all

A neural network with a hundred billion parameters has a loss surface in a hundred-billion-dimensional space. It is non-convex. It has more local minima than there are atoms in the observable universe. It has saddle points uncountable. By any classical optimization argument, descending the gradient with random initial weights and tiny noisy updates should fail completely — get stuck in a bad local minimum, oscillate forever, refuse to converge.

It does not fail. It works. Every time. Modern neural networks routinely train from random initialization to near-state-of-the-art performance, with no special tricks beyond Adam and a learning rate schedule. The reasons are partially understood, frequently surprising, and the subject of ongoing active research.

This entry is shorter than its peers because the answers are *partial*. There is no closed proof. What follows is the current best understanding.

## The classical pessimism

For non-convex optimization in high dimensions, classical theory predicts:

- Local minima everywhere.
- Saddle points everywhere (more abundant in high dimensions).
- Convergence to a poor minimum is the norm.
- Fine-tuning hyperparameters is exponentially expensive.

This was the stated view of optimization theory in the 2000s. Neural-network training was considered a black art, and most theorists predicted it could never work robustly.

It worked anyway.

## The structural surprises

**Local minima are not the problem in high dimensions.**

Dauphin et al. (2014) and others showed empirically and analytically: for random non-convex landscapes in high dimensions, *almost all critical points are saddles*, not minima. A saddle point in dimension \\(d\\) requires \\(d\\) sign choices for the Hessian eigenvalues; a local minimum requires all positive. The probability of a random critical point being a minimum scales as \\(2^{-d}\\). For \\(d \sim 10^9\\), local minima are astronomically rare.

So the optimizer's main risk is not getting stuck in a local minimum (those are vanishingly rare) but slowing down at saddle points (which are everywhere). SGD's noise helps it escape saddles: stochasticity provides random perturbations that knock the iterate off the saddle's unstable directions.

**Most local minima are good enough.**

For overparameterized neural networks (more parameters than training data), the loss landscape has a peculiar property: most local minima have similar loss values, and they are all near zero (training loss approaches zero with sufficient capacity). The minima are connected by low-loss paths in parameter space (mode connectivity, Garipov et al. 2018). So even if SGD finds a "local" minimum, it is essentially as good as any other.

The theory suggests this is a feature of overparameterization. Underparameterized networks have a few sharp, distinct minima with very different qualities; overparameterized networks have a continuum of equivalent solutions, and you land in some part of it.

**SGD finds *flat* minima.**

There are many ways to fit a training set with low loss; the question is which one generalizes. Empirically, SGD has a bias toward *flat* minima — minima where the loss surface is nearly flat in the parameter directions, meaning small parameter perturbations do not change the prediction much.

Flat minima generalize better than sharp ones (Hochreiter-Schmidhuber 1997). The reason: a sharp minimum has a tightly tuned parameter setting that may not transfer to test data (small perturbations produce large changes in output); a flat minimum is robust.

Why does SGD prefer flat minima? Stochastic gradient noise creates an effective Brownian motion that prefers basins where the gradient noise has small variance — typically the flat regions. Recent theory (Chaudhari-Soatto 2018, Smith et al.) characterizes this: SGD's stochastic dynamics is approximately a Langevin diffusion with the loss as potential, and the equilibrium distribution prefers flat minima.

**Implicit regularization (covered separately in this part).**

The gradient descent dynamics itself, in overparameterized settings, has been shown to converge to specific kinds of solutions among the many that fit the training data — typically minimum-norm, maximum-margin, or otherwise well-structured. This is implicit regularization, the topic of a separate entry.

## The "neural tangent kernel" linearization

Jacot, Gabriel, Hongler (2018) showed that infinitely wide neural networks, in the limit, behave like *kernel methods* with a specific kernel (the neural tangent kernel, NTK). In this regime, training is convex (kernel regression) and the dynamics are exactly understood.

For finite-but-large networks, this is an approximation: training sometimes stays in the NTK regime (lazy training, few features change), sometimes leaves it (feature learning, the network re-shapes its representation as it trains).

The NTK theory is a partial answer: in the lazy regime, neural-network optimization is effectively convex, so SGD's success is no longer mysterious. The harder question is why, in the *non-lazy* regime where features actually change, training still works.

## The NTK / mean-field divide

Two complementary theoretical regimes for wide networks:
- **NTK / lazy**: width \\(\to \infty\\), parameters change very little during training, network behavior is well-approximated by a fixed kernel. Loss is effectively convex.
- **Mean-field**: width \\(\to \infty\\), parameters spread out and the network's behavior is described by a probability distribution over neuron states. Loss is non-convex but the dynamics still converge to global minima for shallow networks.

Both predict successful training in the wide limit. Real networks at finite width may sit between these regimes, and a complete theory is still emerging.

## What we still do not understand

- **Initialization matters.** Different initializations give different final losses. Why? Partially explained by the Lipschitz properties of the optimization landscape; not fully.
- **Why Adam beats SGD in some workloads.** Adam's adaptive learning rates are theoretically suspect (they break convergence guarantees), yet empirically dominant in transformer training. The exact mechanism is debated.
- **Generalization of finite-width overparameterized networks.** Why do networks trained to zero training loss often generalize well, when classical learning theory predicts overfitting? Partial answers via implicit regularization, double descent (separate entry), and Rademacher-complexity bounds, but not a clean picture.
- **Loss-landscape phase transitions.** Networks transition between different "phases" during training (memorization vs. generalization, grokking, scaling-law-saturation). These are observed empirically; the theory is in early days.

## The wonder

A method that classical optimization theory said should fail completely, every time, on the kinds of landscapes neural networks have, in fact succeeds at every scale we have tried. The reasons are a combination of:

- Local minima are rare in high dimensions; the optimizer mostly only has to escape saddles.
- Most critical points the optimizer reaches are good enough.
- SGD has a stochastic-dynamics bias toward flat minima, which generalize better.
- Overparameterization (more parameters than data) creates a connected manifold of equivalent solutions.
- In the very-wide limit, networks behave like convex kernel methods.

None of these is a complete answer. Together, they begin to explain why a method that should not work routinely produces models that solve protein folding, generate coherent text, and recognize objects in images.

The wonder is not just that SGD works. It is that the working depends on a confluence of structural facts about high-dimensional non-convex landscapes that no one anticipated when neural networks were first proposed. The "no free lunch" of optimization theory said you should expect failure; the actual landscapes deep networks live on are not the worst-case adversarial ones the no-free-lunch arguments assume. Real-world problems have structure, and the structure is friendly enough to SGD that the method, in practice, just works.

## Where to go deeper

- Bottou, Curtis, Nocedal, *Optimization Methods for Large-Scale Machine Learning*, SIAM Review 2018. Modern optimization perspective.
- Belkin, *Fit Without Fear*, Acta Numerica 2021. The mathematics of why overparameterized models generalize.
