# Markov chain Monte Carlo

You want to draw samples from a probability distribution \\(p(x)\\). You only know \\(p\\) up to a constant — you have a function \\(f(x)\\) such that \\(p(x) \propto f(x)\\), but the normalization \\(\int f\\) is intractable. You also have no closed-form way to invert \\(p\\)'s CDF, so direct sampling is hopeless.

The trick: design a Markov chain whose stationary distribution is \\(p\\), run the chain for a while, and the states it visits are samples from \\(p\\). The chain's transitions depend only on ratios \\(f(x') / f(x)\\), so the unknown normalization cancels. After enough steps, the chain has "mixed" and individual states are approximately drawn from \\(p\\).

This is MCMC. It is the workhorse of Bayesian inference, statistical physics, and large-scale machine learning. Metropolis et al. invented it in 1953 to compute equations of state for hard-sphere fluids; Hastings generalized it in 1970; Gibbs sampling, slice sampling, and Hamiltonian Monte Carlo followed. There is essentially no other practical way to sample from a complicated high-dimensional distribution.

## The Metropolis-Hastings algorithm

Given target distribution \\(p(x) \propto f(x)\\) and a *proposal distribution* \\(q(x' | x)\\) (e.g., a Gaussian centered at \\(x\\)):

```
state x = initial guess
for step = 1, 2, ...:
    propose x' ~ q(x' | x)
    compute acceptance ratio r = (f(x') / f(x)) * (q(x | x') / q(x' | x))
    accept with probability min(1, r); if accept, x = x'
    record x as a sample
```

The chain transitions probabilistically, accepting moves to higher-probability regions deterministically and to lower-probability regions probabilistically. Crucially, the test only requires the *ratio* \\(f(x')/f(x)\\), so the unknown normalization constant of \\(p\\) cancels.

For symmetric proposal (\\(q(x'|x) = q(x|x')\\)), the ratio simplifies to \\(r = f(x')/f(x)\\). This is the basic *Metropolis algorithm*.

## Why it converges

Detailed balance: the constructed chain satisfies \\(p(x) T(x \to x') = p(x') T(x' \to x)\\), where \\(T\\) is the transition kernel (proposal × acceptance). This is a sufficient condition for \\(p\\) to be the stationary distribution of the chain.

Combined with irreducibility (chain can reach any state from any other) and aperiodicity (chain doesn't cycle), the ergodic theorem gives: for any initial state, the empirical distribution of states converges to \\(p\\) as the number of steps goes to infinity. So averages computed from chain states approximate expectations under \\(p\\).

## Why this is unreasonable

Consider what just happened. We have a distribution \\(p\\) we cannot directly sample from. We design a random walk that occasionally accepts and occasionally rejects steps, based on a ratio that requires no global knowledge of \\(p\\). After many steps, the random walk's history *is* a sample from \\(p\\), in the sense that empirical averages converge.

The MCMC user does not know what \\(p\\) actually looks like. They cannot draw it, integrate it, find its mode, or compute its variance directly. But they can compute averages over it, by running a stochastic process whose only inputs are the unnormalized density values at proposed states.

## Gibbs sampling

A specific MCMC for distributions over multiple variables \\(p(x_1, x_2, \dots, x_n)\\). Cycle through variables, sampling each from its conditional distribution given the current values of all others:

```
for i = 1 to n:
    sample x_i from p(x_i | x_1, ..., x_{i-1}, x_{i+1}, ..., x_n)
```

If conditional distributions are easy (closed form, samplable directly), Gibbs avoids the accept/reject step. The resulting chain has \\(p\\) as its stationary distribution (no special proof needed — the conditionals exactly match \\(p\\)).

Used heavily in graphical models, latent Dirichlet allocation (LDA), Bayesian networks. Famously, the Gibbs sampler for the Ising model lets you simulate magnetism on a lattice, leading to the Metropolis-Hastings algorithm in its original 1953 incarnation.

## Hamiltonian Monte Carlo

For continuous distributions, naive Metropolis-Hastings has poor scaling: small steps are slow, large steps get rejected, and high-dimensional distributions are hard to explore by random walk.

Hamiltonian Monte Carlo (HMC; Duane et al., 1987) augments \\(x\\) with a momentum variable \\(p\\) and simulates Hamiltonian dynamics. The "energy" is \\(-\log p(x) + p^T p / 2\\). Steps follow Hamilton's equations (using leapfrog integration); the result moves a long distance per step (because momentum carries the chain through the distribution) while preserving the target via accept/reject for integration error.

HMC dramatically outperforms basic Metropolis-Hastings for high-dimensional smooth distributions. The No-U-Turn Sampler (NUTS; Hoffman and Gelman, 2014) auto-tunes HMC parameters, making it usable as a black box. NUTS is the default in Stan, PyMC, NumPyro — basically every modern Bayesian inference framework.

## Mixing time

The "mix" question — how long until the chain's distribution is close to \\(p\\) — is the critical engineering parameter. For well-behaved distributions, mixing time is polynomial in dimension. For pathological distributions (multi-modal with separated modes, narrow valleys), mixing can be exponentially slow: the chain gets stuck in one mode and rarely jumps.

Diagnostics (effective sample size, R-hat) try to estimate whether the chain has mixed. They are imperfect; insufficient mixing produces wildly wrong inferences with no obvious symptom. Modern Bayesian practice involves running multiple chains from different starting points and comparing their statistics.

For mixing-hard distributions, advanced techniques: parallel tempering (run multiple chains at different "temperatures" and swap), simulated tempering, replica exchange, sequential Monte Carlo. Each tries to bridge separated modes.

## Why this is unreasonable, restated

You have a target distribution. You want to compute expectations. You design a chain that randomly walks around and *converges to the target*, not by knowing the target's shape but by responding to local gradients of \\(\log p\\). After enough steps, the chain's *trajectory* is a sample from the distribution.

It is intuition-violating that a process whose only sensor is "compare \\(f(x)\\) at a few points" can produce samples from a distribution with no other access. The detailed-balance argument is what justifies it: setting up the transition probabilities so that \\(p\\) is invariant. The ergodic theorem says that's enough — invariance implies the chain converges (under mild conditions) to \\(p\\), period.

## Where it shows up

- **Bayesian inference**: posterior distributions are usually intractable except by MCMC. Stan, PyMC, JAGS — every major Bayesian tool implements MCMC. Computational cost is the main bottleneck.
- **Statistical physics**: simulating equilibrium states of fluids, magnets, lattice models. The original use case.
- **Machine learning**: training Boltzmann machines, sampling from energy-based models, pretraining diffusion models (which use related stochastic-differential-equation tools).
- **Phylogenetics**: reconstructing evolutionary trees from DNA data via MCMC over tree space (BEAST, MrBayes).
- **Cryptanalysis**: finding decryption keys by sampling from the posterior over key candidates given a partial plaintext model.

## The wonder

A random walk, where you only ever see local density ratios, samples from any distribution you can describe. The construction is short; the proof of correctness is short; and the technique solves problems that, before its invention, were genuinely intractable.

Bayesian statistics existed before MCMC, but it was an academic curiosity — for any nontrivial model, the posterior was a multidimensional integral that no one could compute. MCMC turned Bayesian inference into something practitioners could actually do, by giving up on direct integration and substituting a stochastic walk that converges to the right answer "asymptotically." The asymptotic-ness is the catch: you never know exactly when you have converged. But empirically, the chains run, the averages stabilize, and the predictions work.

The wonder is in the trade. You give up exact integration; you accept stochastic samples; you let the chain wander; and out the other end, statistical inference at scale.

## Where to go deeper

- Brooks, Gelman, Jones, Meng, eds. *Handbook of Markov Chain Monte Carlo* (CRC, 2011). The reference.
- MacKay, *Information Theory, Inference, and Learning Algorithms*, Chapters 29-33. Free online, beautifully clear.
- Betancourt, *A Conceptual Introduction to Hamiltonian Monte Carlo*, arXiv 2017. The geometric intuition behind HMC.
