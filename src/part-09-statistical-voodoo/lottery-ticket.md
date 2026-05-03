# The lottery ticket hypothesis

You take a fresh, randomly initialized neural network. Train it. Identify the 10% of weights that ended up most important; throw away the other 90%, and reset those 10% back to their *original initialization values*. Now train just that subnetwork, with those original initial values restored, on the same data. It trains to the same accuracy as the full network. The dense network's success was, in some sense, the success of one of its subnetworks: a "winning ticket" that was already present in the random initialization, which the training process found and amplified.

Frankle and Carbin (2018) demonstrated this empirically. The result has held up across model classes, datasets, and pruning methods. It hints that overparameterization is not really about needing many parameters — it is about needing many *candidate subnetworks*, one of which happens to be lucky.

## The procedure

1. Train a dense network \\(N\\) to convergence.
2. Prune the smallest-magnitude weights — say, 90% of them. Keep the surviving 10%.
3. *Rewind* the surviving weights to their initialization values (the random values they had at step 0).
4. Retrain the pruned network from these rewound values.
5. The retrained sparse network achieves accuracy comparable to the dense one.

Crucially, step 3 is the surprise. If you pruned and *re-randomized* the survivors, retraining typically fails. The specific initial values of the surviving weights matter; they were lucky. The pruning mask plus the rewound initial values constitute the "winning ticket."

## The original empirical result

Frankle and Carbin tested on small networks (LeNet on MNIST, VGG-style on CIFAR-10). For each:

- Train, prune by magnitude, rewind, retrain. Achieves 90%+ of original accuracy at 90% sparsity, or 80% accuracy at 99% sparsity, depending on architecture.
- Compare with random masks at same sparsity (re-init the survivors): much lower accuracy.

The result was striking enough to spawn a research subfield. Subsequent work (Frankle et al. 2019, Yu et al. 2019) confirmed it for ResNets and larger models, with a refinement: for big networks, you may need to rewind not to step 0 but to step ~1000, called *late rewinding*.

## What this implies

The dominant interpretation: a randomly initialized large network contains, with high probability, many trainable subnetworks. Training a dense network is, partly, a search for one such subnetwork to propagate. The *function* of overparameterization is to provide enough random initializations that at least one subnetwork is well-positioned.

If true, this changes the story about why neural networks work:

- *It is not about needing many parameters.* It is about needing many independent random initializations of a small network.
- *Overparameterization is a search-space increase.* More parameters means more chances of a lucky subnetwork.
- *Training is partly identification, partly amplification.* Training picks out the lucky subnetwork (via gradient flow concentrating on it) and amplifies its weights, suppressing the rest.

This connects to *strong lottery tickets* (Ramanujan et al. 2019, Malach et al. 2020): for sufficiently overparameterized networks, the *random initialization itself* contains a subnetwork (no training needed!) that achieves the desired performance. You can find a high-accuracy subnet by simply *masking* — choosing which weights to use — without changing any weight values. Empirically demonstrated; theoretically, sufficient overparameterization does guarantee this.

## The strong-lottery-ticket theorem

Malach, Yehudai, Shalev-Shwartz, Shamir (2020): for sufficiently large depth and width, *any* target function expressible by a neural network can be approximated by an appropriate subnetwork of a randomly initialized larger network — without training. Just by selecting which weights to "use."

The proof is by counting: the number of distinct subnetworks of a random network is exponential in its size; for any target, with high probability some subnetwork is close to it. The construction is non-explicit but the existence theorem is proved.

This is a strong form of the lottery-ticket hypothesis: even *no* training, just masking, suffices. In practice, you need a way to find the mask, and that takes some training of a "supermask" (a learned set of binary mask values). The amount of compute to find the mask can be smaller than what you would spend to train weights directly.

## What it does not explain

- *Why is the mask findable by gradient descent?* The lottery ticket hypothesis says the ticket exists; the practical question is whether magnitude pruning (or any other heuristic) reliably finds it. Empirically yes for moderate networks; sometimes less reliably for very large ones.
- *Why does late rewinding work better for large networks?* Hypothesis: very early in training, weight signs stabilize and the magnitudes start to differentiate; rewinding to this slightly-trained state preserves the sign pattern that makes the ticket work.
- *Are tickets transferable across tasks?* Sometimes, partially. Tickets identified on one task often work on related tasks but not unrelated ones.

## Practical impact

Lottery-ticket-style pruning is now a standard ingredient in neural-network compression. The recipe:

1. Train dense.
2. Prune.
3. (Optionally rewind.)
4. Retrain.

Achieves 90%+ sparsity with minimal accuracy loss for many production models. Useful for deploying big models on resource-constrained devices.

Inference-only sparsity (use a sparse model at deployment, but train a dense one) is widespread. Training-time sparsity (train only the sparse model) is harder but is being deployed for very large models where dense training is too expensive.

## What this says about training

The lottery-ticket framing reframes "training" as approximately:

- The initialization randomly seeds many candidate subnetworks.
- Training, via gradient descent and the loss landscape, identifies and amplifies one (or a few) good subnetworks.
- The amplified subnet does the actual computation; the others slowly fade or contribute background noise.

Under this view, overparameterization is exploration in initialization space. Bigger networks explore more candidates; by the law of large numbers, they are more likely to contain a good ticket.

## The wonder

A randomly initialized neural network already contains, hidden inside it, the subnetwork that will eventually do the work — at least up to the precise initial weight values. Training does not so much "build" the right computation as "find" it among the many random possibilities the initialization provides.

This is a startlingly different picture from the classical "neural networks learn by adjusting weights from random toward the right values." In the lottery-ticket frame, the right values were already there; the network just had a lot of *wrong* values surrounding them. Training discards the wrong values (effectively zeroes them) and amplifies the right ones.

It says something deep about the nature of overparameterized learning. The capacity of a large network is not "in the weights" but rather "in the space of subnetworks." Most of the parameters are noise. A small lucky fraction does the work. The dense network is a *combinatorial substrate* — it provides the combinatorial space of possible computations from which gradient descent selects.

If this is right (and the empirical evidence for the basic version is strong), then the long argument about whether "neural networks really learn" or "merely memorize" or "interpolate" misses the point. They identify-and-amplify. The learning is in the initialization plus the selection.

## Where to go deeper

- Frankle and Carbin, *The Lottery Ticket Hypothesis: Finding Sparse, Trainable Neural Networks*, ICLR 2019. The original.
- Malach, Yehudai, Shalev-Shwartz, Shamir, *Proving the Lottery Ticket Hypothesis: Pruning is All You Need*, ICML 2020. The strong-lottery-ticket theorem.
