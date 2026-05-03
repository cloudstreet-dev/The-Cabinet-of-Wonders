# Variational inference

You have an intractable posterior distribution \\(p(z | x)\\). MCMC samples from it stochastically, slowly. Variational inference (VI) takes a different approach: pick a tractable family of distributions \\(q_\phi(z)\\) — say, factorized Gaussians — and *optimize* the parameters \\(\phi\\) so that \\(q_\phi\\) is as close as possible to \\(p(z | x)\\). What was a sampling problem becomes an optimization problem, with all the standard gradient-based machinery.

The result is approximate, but fast. VI scales to massive datasets where MCMC chokes. Modern variational autoencoders, Bayesian neural networks, and many large-scale generative models rely on VI as their inference engine.

The genuinely strange part: by minimizing a divergence between approximation and target, VI lets you do Bayesian inference in cases where the target distribution cannot even be evaluated point-wise (only its log-density up to a constant). Like MCMC, the unknown normalizer drops out of the math.

## The setup

Posterior: \\(p(z | x) = p(x, z) / p(x)\\) where \\(p(x) = \int p(x, z) dz\\) is the marginal likelihood (the troublesome integral).

Choose a family \\(q_\phi(z)\\) — say, \\(q_\phi(z) = \mathcal{N}(z; \mu, \text{diag}(\sigma^2))\\) parameterized by \\(\phi = (\mu, \sigma)\\).

Goal: find \\(\phi^*\\) minimizing the *Kullback-Leibler divergence* \\(\text{KL}(q_\phi(z) \| p(z | x))\\).

Direct minimization requires evaluating \\(p(z | x)\\), which we cannot. The trick:

\\[ \text{KL}(q \| p) = \int q(z) \log \frac{q(z)}{p(z | x)} dz \\]
\\[ = \mathbb{E}_q[\log q(z) - \log p(z | x)] \\]
\\[ = \mathbb{E}_q[\log q(z) - \log p(x, z) + \log p(x)] \\]
\\[ = \log p(x) - \mathbb{E}_q[\log p(x, z) - \log q(z)] \\]
\\[ = \log p(x) - \text{ELBO}(\phi) \\]

where the *Evidence Lower BOund* (ELBO) is

\\[ \text{ELBO}(\phi) = \mathbb{E}_{q_\phi}[\log p(x, z) - \log q_\phi(z)] \\]

\\(\log p(x)\\) is a constant in \\(\phi\\), so minimizing KL is equivalent to *maximizing* the ELBO. And the ELBO requires only the joint \\(p(x, z)\\), which we *can* compute, plus expectations under \\(q\\), which we can sample from.

So we have an objective we can compute and differentiate: maximize ELBO with respect to \\(\phi\\), and \\(q_\phi\\) approaches the true posterior.

## Mean-field VI

The classical version of VI assumes a factorized approximation:

\\[ q(z) = \prod_i q_i(z_i) \\]

Each variable's distribution is independent. The optimal factor for each \\(z_i\\), holding others fixed, is

\\[ q^*_i(z_i) \propto \exp\left(\mathbb{E}_{q_{-i}}[\log p(x, z)]\right) \\]

This gives an iterative algorithm: cycle through factors, computing each as the exponential of the expected joint log-density.

For conjugate models (where the conditional distribution of each variable, given the others, has the same family as the prior), the updates are closed-form. For example, in a Gaussian mixture model with Gaussian priors, all updates are explicit.

Mean-field is fast but limited: by assuming independence between factors, it cannot capture correlations between latent variables. The approximation is often biased — variances are typically underestimated.

## Black-box VI and the reparameterization trick

For more flexibility, drop the conjugacy assumption. The challenge: gradients of \\(\text{ELBO}(\phi)\\) involve \\(\nabla_\phi \mathbb{E}_{q_\phi}[\log p(x, z) - \log q_\phi(z)]\\), and the expectation depends on \\(\phi\\), so \\(\nabla_\phi\\) cannot be moved inside the expectation directly.

Two solutions.

**Score function (REINFORCE) gradient**: \\(\nabla_\phi \mathbb{E}_{q_\phi}[f(z)] = \mathbb{E}_{q_\phi}[f(z) \nabla_\phi \log q_\phi(z)]\\). Estimable by sampling. Variance is high; needs control variates.

**Reparameterization trick** (Kingma-Welling, 2013): if \\(z = g_\phi(\epsilon)\\) for some auxiliary noise \\(\epsilon\\) (e.g., \\(z = \mu + \sigma \cdot \epsilon\\) with \\(\epsilon \sim \mathcal{N}(0, I)\\)), then \\(\mathbb{E}_{q_\phi}[f(z)] = \mathbb{E}_\epsilon[f(g_\phi(\epsilon))]\\). The expectation is now over a fixed distribution; gradients can pass through \\(g_\phi\\) directly, low-variance, no special tricks.

The reparameterization trick is so important that the next entry in this part is dedicated to it.

## Variational autoencoders

The VAE (Kingma-Welling, 2013): a generative model with a flexible learned encoder \\(q_\phi(z | x)\\) (a neural network mapping \\(x\\) to the parameters of \\(q\\)) and a learned decoder \\(p_\theta(x | z)\\) (another neural network). Training maximizes the ELBO:

\\[ \mathcal{L}(\theta, \phi; x) = \mathbb{E}_{q_\phi(z|x)}[\log p_\theta(x | z)] - \text{KL}(q_\phi(z|x) \| p(z)) \\]

The first term is a reconstruction loss (encode \\(x\\) to \\(z\\), decode back to \\(x\\)); the second is a regularizer pulling \\(q_\phi\\) toward a simple prior \\(p(z)\\), typically standard normal.

The reparameterization trick makes the gradients tractable. Training is end-to-end backprop. The result: a generative model where you can sample from \\(p(z)\\) (easy: standard normal) and decode to a sample from approximately \\(p(x)\\). For images, this gives a model that can generate novel samples.

VAEs are now standard, alongside diffusion models, GANs, and autoregressive models, in generative modeling. Their crispness vs. fidelity trade-off (more expressive \\(q\\) families give better likelihoods but blurrier samples) drives ongoing research.

## Why this is voodoo

You start with a posterior you cannot compute. You parameterize a *different* family of distributions and minimize their KL divergence to the unreachable posterior — using only what you *can* compute. The optimization problem is well-defined, the gradient signal is real, the algorithm converges, and the resulting approximation is good enough for many applications.

The structural insight: KL divergence to an intractable target reduces, modulo a constant, to a tractable expectation. The intractable bit is the part that does not depend on the variational parameters, so it cancels from the gradient. You are simultaneously sneaking past the intractability barrier and getting a usable optimization.

## Where it falls short

VI is approximate. KL(\\(q\\)||\\(p\\)) is asymmetric — the optimum in this direction is *mode-seeking*: the approximation tries to fit one mode well rather than cover all modes. So multimodal posteriors get reduced to one mode. Use cases where you care about all modes (model selection, unbiased predictions) need the reverse direction, KL(\\(p\\)||\\(q\\)), which is much harder to optimize.

VI also tends to underestimate posterior variance — quantitative uncertainty estimates from VI are systematically tighter than the true posterior. For applications where calibrated uncertainty matters (medical decisions, scientific inference), VI's biases are a known issue and a focus of methodological work.

For genuinely correct samples from a complex posterior, MCMC is still the gold standard, despite its computational cost. VI's appeal is throughput.

## The wonder

You convert an intractable Bayesian inference into a tractable optimization. The trick is the ELBO: a function of the variational parameters that *can* be computed (because the unknown normalizer cancels) and is a lower bound on the marginal likelihood (which we cannot compute either, but whose optimization is equivalent to KL minimization). Maximize the ELBO, and you have approximated the posterior. With the reparameterization trick, this becomes plain old gradient descent.

Variational inference is the move that lets Bayesian models scale to billion-parameter neural networks. MCMC cannot run on those models in any reasonable time. VI runs in seconds per epoch, with the same backprop infrastructure as standard deep learning. The cost is approximation; the win is feasibility.

## Where to go deeper

- Blei, Kucukelbir, McAuliffe, *Variational Inference: A Review for Statisticians*, JASA 2017. Modern review.
- Kingma and Welling, *Auto-Encoding Variational Bayes*, ICLR 2014. The VAE paper.
