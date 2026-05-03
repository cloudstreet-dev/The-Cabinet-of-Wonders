# Implicit regularization

A classical learning-theory argument: a model with more parameters than training examples will overfit. The model has the capacity to memorize the training data exactly, including its noise. Test performance should be terrible. Add regularization (L2 penalty, dropout, weight decay) to limit capacity, and you might generalize.

Modern deep learning ignores this prediction. Networks with billions of parameters trained on millions of examples — far overparameterized by the classical accounting — generalize remarkably well, often *without* explicit regularization beyond modest weight decay. The classical theory was wrong about something fundamental, and the missing ingredient is *implicit* regularization: the optimizer itself, by following gradient descent dynamics, implicitly biases the solution toward "well-behaved" parameter values, even though no penalty term explicitly encodes that bias.

## The setup

A neural network has many parameters \\(\theta\\). Training data fits a set of constraints: for each example, the network's output should be close to the target. With enough parameters, the constraints are *underdetermined* — there are many \\(\theta\\) that fit the training data perfectly. Among those many, which one does gradient descent select?

The constraint manifold is high-dimensional; typical points on it can have wildly different generalization behavior. Some satisfy the training constraints by memorizing noise; some by learning the underlying function. Generalization depends on which.

Implicit regularization: gradient descent, by its dynamics, prefers certain manifold points over others. The preference is *implicit* — there is no penalty term saying "prefer this solution"; the optimizer just lands at certain solutions naturally.

## Linear regression: the cleanest example

Consider overparameterized linear regression. Find \\(\theta \in \mathbb{R}^d\\) such that \\(X \theta = y\\) where \\(X \in \mathbb{R}^{n \times d}\\) with \\(d > n\\). Many solutions exist.

Gradient descent on \\(\| X \theta - y \|^2\\) starting from \\(\theta_0 = 0\\) converges (under mild conditions) to the *minimum-norm solution*: \\(\theta^* = X^+ y\\), where \\(X^+\\) is the Moore-Penrose pseudoinverse. This is the \\(\theta\\) closest to the origin among those that fit the data.

The minimum-norm solution is what \\(\ell_2\\) regularization would produce in the limit \\(\lambda \to 0\\). So gradient descent from zero gives the same answer as ridge regression with vanishing regularization. The "regularization" is implicit in the dynamics, not in the loss function.

This is a clean theorem, and the picture generalizes to other settings.

## Logistic regression and margin maximization

Another clean case: logistic regression on linearly separable data. The training loss can be made arbitrarily small by scaling the parameter vector to infinity. So the loss landscape has a "trough" extending to infinity along certain directions.

Soudry et al. (2018): gradient descent on this trough drifts in the direction of *maximum margin*. As training proceeds and parameter norms grow, the *direction* of \\(\theta\\) converges to the maximum-margin SVM solution.

The SVM solution is the maximum-margin separator; it is the unique direction that maximizes the smallest training-point distance from the decision boundary. Gradient descent finds this without an explicit margin objective. Implicit regularization toward maximum margin.

## In neural networks: less clean but still real

For deep neural networks, characterizing the implicit regularization is harder. Some empirical and theoretical findings:

**Frequency bias.** Neural networks fit low-frequency components of the target function before high-frequency. So during training, simple functions are learned first, complex ones later. If you stop training early, you get a simpler function — implicit early-stopping regularization.

**Flat-minima preference.** SGD's stochastic noise biases the optimizer toward flat minima of the loss surface. Flat minima generalize better than sharp ones (a perturbation in parameter space changes predictions less for flat minima). The connection between SGD noise structure and flatness preference has been formalized for various noise models.

**Maximum-margin in deep ReLU networks.** Lyu and Li (2020), Chizat and Bach (2020): for homogeneous neural networks (ReLU networks satisfying \\(f(\alpha \theta) = \alpha^L f(\theta)\\) for some degree \\(L\\)), gradient descent on logistic loss converges in direction to a KKT point of a max-margin problem in function space. The exact characterization is complex, but the upshot: deep networks, like their linear analogs, end up at maximum-margin solutions.

**Sparse and low-rank biases.** In matrix factorization (low-rank approximation by gradient descent on \\(\|UV^T - M\|^2\\)), the optimizer implicitly prefers low-rank solutions. Even with no explicit rank constraint, gradient flow finds low-rank \\(UV^T\\) when one exists.

## Why this is voodoo

There is no principled reason gradient descent should be a *good* algorithm for finding generalizing solutions among the many that fit the training data. It is just one optimization algorithm; the loss surface has no preference among the perfect-fit solutions.

Yet gradient descent reliably finds *good* solutions. The bias toward "minimum-norm" or "maximum-margin" or "low-rank" or "low-frequency-first" all turn out to be exactly the right priors for generalization on natural data.

Why does the optimizer's bias align with what generalizes? This is the deep mystery. Several partial answers:

- **The bias is a regularization that physically corresponds to "simple," and natural data has the property that simple-fitting solutions also generalize.**
- **Gradient descent dynamics has a smoothness property that aligns with the function-space geometry.**
- **The architectural choices (ReLU, layer depth) interact with the dynamics in ways that produce useful biases.**

Each is partially true. None is a complete theory.

## The double-descent connection

The traditional bias-variance picture predicts a U-shaped error curve: low capacity has high bias, high capacity has high variance, optimal is in the middle. *Double descent* (covered separately) shows the curve goes back down: as capacity grows past the interpolation threshold, test error decreases again, often to better levels than the classical optimum. Implicit regularization is part of why: in the overparameterized regime, the optimizer's bias selects "good" solutions among the many that fit.

## Why it matters for practice

If implicit regularization is doing the work, then explicit regularization (weight decay, dropout, L2 penalty) is *additive* — adjusting the implicit bias rather than the only source of regularization. Empirically, modest weight decay improves generalization but is not necessary for it.

The choice of optimizer matters more than classical theory predicted. Adam vs. SGD vs. heavy-ball momentum all have different implicit biases. Empirically, SGD often generalizes better than Adam on classification tasks (and Adam often trains faster); the explanation is that SGD's noise-induced bias toward flat minima is more aligned with what generalizes.

## Where this leaves theory

For decades, learning theory said: the right way to control overfitting is by limiting model capacity (VC dimension, Rademacher complexity, etc.). Modern deep learning showed that *capacity* is not the bottleneck for generalization in overparameterized models; the *optimizer's implicit biases* are. The right theory is being built around how those biases interact with data structure to produce learners that generalize despite having far more capacity than the data should support.

The 2020s research on implicit regularization is one of the most active in machine learning. Each year, new biases are characterized for new model classes. The picture is filling in.

## The wonder

The optimizer is supposed to be a means, not an end. It finds a minimum; the minimum's quality is up to the loss function and architecture. Modern deep learning is showing this naive division is wrong: the optimizer is *also* a regularizer. Its dynamics select among the many minima that fit the training data, and the selection happens to be one that generalizes.

That this works at all is the wonder. Gradient descent, the simplest optimization algorithm, was not designed to do anything more than minimize loss. In overparameterized settings it does much more: it picks the *right* loss minimum. The picking is encoded entirely in the dynamics — momentum, step size, batch size, noise structure — without any explicit regularization term. Tweak the optimizer, you tweak the bias. Modern hyperparameter search is, partially, the search for an optimizer with the right implicit prior for the data at hand.

## Where to go deeper

- Soudry, Hoffer, Nacson, Gunasekar, Srebro, *The Implicit Bias of Gradient Descent on Separable Data*, JMLR 2018. The clean linear case.
- Belkin, *Fit Without Fear*, Acta Numerica 2021. The mathematics of the modern picture.
