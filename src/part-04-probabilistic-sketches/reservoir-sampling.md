# Reservoir sampling

You are reading a stream of items whose total length you do not know in advance. You want to maintain, at all times, a uniformly random sample of \\(k\\) items from everything you have seen so far. The trick takes constant memory (an array of size \\(k\\)) and constant time per item, and at the end of an arbitrary-length stream the sample is exactly uniform — every \\(k\\)-subset of the seen items is equally likely.

The algorithm is one of the cleanest examples of a streaming primitive whose analysis is genuinely surprising the first time you see it. Knuth attributes it to "Alan Waterman."

## The algorithm

```
fill the reservoir R[0..k-1] with the first k items from the stream
for i = k+1, k+2, k+3, ... (subsequent items):
    pick a uniform random integer j in [1, i]
    if j <= k:
        R[j-1] = current item
```

That is the whole algorithm. After processing \\(n\\) items, every item from the stream is in the reservoir with probability exactly \\(k/n\\), and any specific subset of \\(k\\) items appears with probability \\(\binom{n}{k}^{-1}\\).

## Why it works

Let \\(P_n(i)\\) denote the probability that item \\(i \in \{1, \dots, n\}\\) is in the reservoir after \\(n\\) items have been processed. Claim: \\(P_n(i) = k/n\\) for every \\(i \leq n\\).

Inductive proof. Base case \\(n = k\\): every item is in the reservoir, \\(P_k(i) = 1 = k/k\\).

Inductive step \\(n \to n + 1\\):

For item \\(n+1\\) (the new arrival): chosen with probability \\(k/(n+1)\\) (since the algorithm picks \\(j \in \{1, \dots, n+1\}\\) and includes the new item iff \\(j \leq k\\)).

For an old item \\(i \leq n\\): it must have been in the reservoir at step \\(n\\) (probability \\(k/n\\) by induction) and not have been evicted by the new arrival (probability \\(1 - 1/(n+1)\\), because eviction happens with probability \\(1/(n+1)\\) for any specific reservoir slot). So

\\[ P_{n+1}(i) = \frac{k}{n} \cdot \left( 1 - \frac{1}{n+1} \right) = \frac{k}{n} \cdot \frac{n}{n+1} = \frac{k}{n+1} \\]

Both cases give \\(k/(n+1)\\). Induction closes.

## Why uniform-marginals isn't enough

A stronger property is uniform over \\(k\\)-subsets — not just that each item is in with probability \\(k/n\\), but that any specific \\(k\\)-tuple of items is the reservoir with the same probability \\(\binom{n}{k}^{-1}\\). Reservoir sampling has this property too. Proof: similar induction on the joint probability. The marginals matching is necessary but not sufficient; the algorithm gives the joint as well, because at each step the eviction is uniform over reservoir slots.

## Why this is the only natural algorithm

Imagine you tried something different: store every item, then at the end pick \\(k\\) at random. That requires \\(O(n)\\) memory, and you cannot output anything until the stream ends. Disqualified for streams of unknown or unbounded length.

Imagine you tried: keep every \\(n/k\\)-th item. That requires knowing \\(n\\) in advance.

Imagine you tried: independently for each item, include it with probability \\(k/n\\) where \\(n\\) is the current count. The reservoir size becomes a random variable with mean \\(k\\) but high variance — sometimes small, sometimes huge. No good.

Reservoir sampling threads the needle: constant memory, exactly uniform, no advance knowledge of stream length. It is the algorithm.

## Weighted reservoir sampling

Sometimes items have weights and you want the sample to reflect the weights — high-weight items should be more likely to appear.

The Algorithm A-Res (Efraimidis-Spirakis 2006): for each item with weight \\(w_i\\), generate a key \\(u_i = r_i^{1/w_i}\\) where \\(r_i\\) is uniform on \\([0, 1]\\). Maintain the reservoir as the \\(k\\) items with the largest keys, which can be done with a min-heap.

The math: the largest of \\(n\\) such keys corresponds, in expectation, to the items selected by *exponential clocks* with rates \\(w_i\\), which is exactly the right distribution for weighted random sampling without replacement. Each insertion is \\(O(\log k)\\); total memory is \\(O(k)\\); single-pass.

## Distributed reservoir sampling

You have data on multiple machines. Each machine has produced a reservoir of size \\(k\\) over its own stream. How to combine?

Naive: take the union (\\(km\\) items for \\(m\\) machines), discard duplicates, sample. But the per-machine reservoirs are biased toward shorter streams (each item from a stream of length \\(n\\) appears with probability \\(k/n\\), not \\(k / \sum_j n_j\\)).

Correct: each machine reports its reservoir along with the count of items it processed (\\(n_j\\)). The combiner samples from each reservoir with probability proportional to \\(n_j\\) — items in machine \\(j\\)'s reservoir each represent a "fair share" of \\(n_j\\) items from the global stream. Or, equivalently, treat each machine's reservoir as a weighted item set and use weighted reservoir sampling on top.

This is how distributed analytics systems compute uniform samples for downstream querying without ever materializing the full data set.

## Where it shows up

- **Database query sampling.** "Give me a uniform random 10000 rows from this 10-billion-row table." Reservoir sampling on a single pass.
- **Log analysis.** Maintain a uniform sample of recent log lines for human inspection without buffering the whole log.
- **A/B testing on streams.** Maintain unbiased samples of user events from a high-volume stream.
- **Game/simulation telemetry.** Sample player actions from millions per second for offline analysis.
- **Random k-out-of-n choice in interview problems.** "Randomly choose a line from a file of unknown length" — reservoir sampling, \\(k = 1\\).

## The \\(k = 1\\) case

For \\(k = 1\\), the algorithm is: when the \\(i\\)-th item arrives, replace the current sample with probability \\(1/i\\). After processing \\(n\\) items, the sample is uniform over the stream.

The proof in this case is one line: \\(P(\text{item } i \text{ is final sample}) = \frac{1}{i} \cdot \prod_{j=i+1}^{n} \left(1 - \frac{1}{j}\right) = \frac{1}{i} \cdot \frac{i}{n} = \frac{1}{n}\\). The product telescopes — each term \\(1 - 1/j = (j-1)/j\\), and the product from \\(j = i+1\\) to \\(n\\) of \\((j-1)/j\\) collapses to \\(i/n\\).

This is the "random line of a file" interview question. The answer is two-line bash:

```sh
awk 'rand()*NR < 1 { line = $0 } END { print line }' file.txt
```

## The wonder

The algorithm runs in constant memory. Its output is exactly uniform — no approximation, no failure probability. It works on a stream of unknown length. The proof is one inductive paragraph. Once you understand it, the next time you encounter a stream-sampling problem, you know exactly what to do.

The reason it feels surprising is that the intuition pulls in the wrong direction. Most people, faced with "sample uniformly from an unknown-length stream," want to wait until the end. The algorithm shows that patience is unnecessary; you can maintain a uniform sample at every prefix, with O(1) work per item, by choosing the eviction probabilities exactly right.

## Where to go deeper

- Vitter, *Random Sampling with a Reservoir*, ACM TOMS 1985. The classical reference, including faster algorithms (skip-counting) for large streams.
- Efraimidis and Spirakis, *Weighted Random Sampling with a Reservoir*, IPL 2006. The weighted variant.
