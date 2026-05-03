# The reparameterization trick

You want to backpropagate through a random sample. You have a function that depends on the parameters of a probability distribution; you want gradients of an expectation under that distribution; you want to use ordinary backprop. But sampling is not differentiable: \\(z \sim \mathcal{N}(\mu, \sigma)\\) is a stochastic operation; you cannot push a gradient through it.

The reparameterization trick rewrites the sample as a deterministic transformation of fixed-distribution noise: \\(z = \mu + \sigma \cdot \epsilon\\) with \\(\epsilon \sim \mathcal{N}(0, 1)\\). Now the only stochastic input is \\(\epsilon\\), whose distribution does not depend on the parameters. The path from \\(\mu, \sigma\\) to \\(z\\) is fully differentiable. Gradients flow.

The trick is one line of math. It is the secret behind why VAEs train at all, and is part of why almost every modern probabilistic deep-learning method works.

## The problem

Consider an expectation under a parameterized distribution:

\\[ \mathcal{L}(\phi) = \mathbb{E}_{q_\phi(z)}[f(z)] \\]

You want \\(\nabla_\phi \mathcal{L}\\). Naive Monte Carlo: sample \\(z \sim q_\phi\\), evaluate \\(f(z)\\), backprop. The problem: the sampling step \\(z \sim q_\phi(z)\\) is stochastic and not in any obvious sense differentiable.

You can compute the gradient using the *score function* (REINFORCE) estimator:

\\[ \nabla_\phi \mathcal{L} = \mathbb{E}_{q_\phi}[f(z) \nabla_\phi \log q_\phi(z)] \\]

This works but has high variance. Hundreds or thousands of samples per step. For neural networks with millions of parameters, this is too slow.

## The trick

If \\(z\\) can be written as a deterministic function of fixed-distribution noise \\(\epsilon\\):

\\[ z = g_\phi(\epsilon), \quad \epsilon \sim p(\epsilon) \\]

(where \\(p(\epsilon)\\) does not depend on \\(\phi\\)), then

\\[ \mathbb{E}_{q_\phi(z)}[f(z)] = \mathbb{E}_{p(\epsilon)}[f(g_\phi(\epsilon))] \\]

The right side has fixed-distribution noise. Gradients of this expectation are clean:

\\[ \nabla_\phi \mathbb{E}_{p(\epsilon)}[f(g_\phi(\epsilon))] = \mathbb{E}_{p(\epsilon)}[\nabla_\phi f(g_\phi(\epsilon))] \\]

Sample \\(\epsilon\\), compute \\(z = g_\phi(\epsilon)\\), evaluate \\(f(z)\\), backprop through \\(g_\phi\\) to get gradients with respect to \\(\phi\\). One sample per step is enough; gradients are low-variance.

For a Gaussian: \\(z = \mu + \sigma \cdot \epsilon\\), \\(\epsilon \sim \mathcal{N}(0, 1)\\). The gradient with respect to \\(\mu\\) is just \\(\partial f / \partial z\\); with respect to \\(\sigma\\) it is \\(\epsilon \cdot \partial f / \partial z\\). Mechanical.

## Why this works mathematically

The key fact: pushforward of a parameter-independent distribution through a parameter-dependent function. Sampling \\(z\\) from \\(q_\phi\\) is the same as sampling \\(\epsilon\\) from \\(p\\) and computing \\(z = g_\phi(\epsilon)\\), as long as \\(g_\phi\\) and \\(p\\) are chosen so that the resulting distribution of \\(z\\) is \\(q_\phi\\). The two are *equivalent* as random variables, but they are *different* as computational graphs: in the first, \\(\phi\\) is buried inside a sampler; in the second, \\(\phi\\) is in a deterministic function with random input.

The deterministic-function representation supports gradients. The sampler representation does not. Same random variable, different computational structure.

## Which distributions admit this

Any *location-scale* family does:
- Gaussian: \\(z = \mu + \sigma \epsilon\\), \\(\epsilon \sim \mathcal{N}(0, 1)\\).
- Uniform: \\(z = a + (b - a) \epsilon\\), \\(\epsilon \sim \text{Unif}(0, 1)\\).
- Laplace: \\(z = \mu + b \epsilon\\), \\(\epsilon \sim \text{Laplace}(0, 1)\\).
- Generally, any distribution closed under affine transformations.

Other reparameterizations:
- Inverse-CDF: \\(z = F^{-1}_\phi(\epsilon)\\), \\(\epsilon \sim \text{Unif}(0, 1)\\). Works whenever \\(F^{-1}\\) is differentiable.
- Implicit: \\(z\\) defined as the solution to an equation involving \\(\epsilon\\) and \\(\phi\\). Implicit-function theorem gives gradients.

For *discrete* distributions (categorical, Bernoulli), reparameterization in the strict sense fails — there is no smooth function from continuous noise to discrete outputs. Workarounds: the Gumbel-softmax trick (Jang et al., 2016) approximates a categorical sample with a continuous "soft" version that is reparameterizable, controlled by a temperature; as the temperature anneals to zero, samples become approximately discrete.

## Where it shows up

- **Variational autoencoders**: the encoder produces \\((\mu, \sigma)\\); the latent \\(z\\) is sampled via the reparameterization trick; the decoder maps \\(z\\) back to \\(x\\). End-to-end backprop trains everything.
- **Stochastic neural networks**: any layer involving sampled randomness — dropout (with continuous variants), Bayesian neural nets, normalizing flows — uses reparameterization.
- **Reinforcement learning**: stochastic policy gradient methods can use reparameterization for continuous action spaces (deterministic policy gradient is one form).
- **Diffusion models**: train by reparameterizing the noise added at each step, getting low-variance gradients.

## Comparison with score-function gradients

The score-function (REINFORCE) gradient does not require reparameterization: it works for any sampler, but has high variance. Variance reduction tricks — control variates, baselines, importance sampling — bring REINFORCE down to usable levels in some applications, but reparameterization (when applicable) is generally orders of magnitude better.

The trade-off: REINFORCE works for arbitrary distributions and even non-differentiable \\(f\\); reparameterization needs a smooth \\(g_\phi\\). For continuous distributions and smooth losses, reparameterization wins. For discrete or non-smooth, REINFORCE or its variants are needed.

## The wonder

A trivial algebraic rewrite — moving the parameter from the sampler to the post-sampling transformation — turns "stochastic computation graph" (which gradient descent cannot easily reach) into "deterministic computation graph with random input" (which it can). The same mathematical content, two different representations, dramatically different computational properties.

This is one of those tricks where the moment you see it, you feel like you should have seen it earlier. The 2013 Kingma-Welling and Rezende-Mohamed-Wierstra papers introduced VAEs and SGVI by leaning on this trick; before then, gradient-based deep generative models were largely impossible. Now they are routine.

The wonder is in the move's smallness: \\(z = \mu + \sigma \epsilon\\). Three symbols of algebra. With it, all of modern deep generative modeling becomes feasible. Without it, you are stuck with REINFORCE, and the field looks completely different.

## Where to go deeper

- Kingma and Welling, *Auto-Encoding Variational Bayes*, ICLR 2014. The reparameterization trick is in section 2.4.
- Rezende, Mohamed, Wierstra, *Stochastic Backpropagation*, ICML 2014. Independent, contemporaneous derivation, with more general reparameterizations.
