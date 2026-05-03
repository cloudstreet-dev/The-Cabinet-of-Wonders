# Linearity of expectation

A combinatorial-counting problem that looks like it requires inclusion-exclusion across thousands of cases collapses to one line of arithmetic, because the average of a sum is the sum of the averages — even when the things you are summing are wildly correlated. That fact, in its simplest form, is taught in the first probability lecture. Its consequences are pulled out of a hat at every level of the subject for the rest of the curriculum.

## The statement

For any random variables \\(X_1, X_2, \dots, X_n\\) on the same probability space:

\\[ E[X_1 + X_2 + \cdots + X_n] = E[X_1] + E[X_2] + \cdots + E[X_n] \\]

That is it. No independence assumed. No identical-distribution assumed. No common probability assumed. The variables can be defined however you like — as long as their expectations exist, the expectation of their sum is the sum of their expectations.

The proof is one line. By definition,

\\[ E[X + Y] = \sum_\omega P(\omega) (X(\omega) + Y(\omega)) = \sum_\omega P(\omega) X(\omega) + \sum_\omega P(\omega) Y(\omega) = E[X] + E[Y] \\]

Linearity of summation, applied inside an integral. Trivial. The trick is what happens when you use it.

## Why the lack of independence is the magic

Most probability identities require independence. \\(E[XY] = E[X] E[Y]\\) requires \\(X \perp Y\\). \\(\text{Var}(X+Y) = \text{Var}(X) + \text{Var}(Y)\\) requires \\(X \perp Y\\). Failure to recognize that an identity needs independence is the most common student mistake in the subject.

Linearity of expectation does not need it. So you can sum up indicators of *highly correlated* events and still get a correct expected count. That is the move that lets you finesse problems that would be intractable by direct case analysis.

## Hat-check problem

\\(n\\) people leave their hats at the door and pick one up uniformly at random when they leave. What is the expected number who get their own hat back?

Direct approach: enumerate over all \\(n!\\) permutations, count fixed points, divide by \\(n!\\). For large \\(n\\), this is a derangement-counting problem. Possible, but you need inclusion-exclusion.

Linearity approach: let \\(X_i\\) be the indicator that person \\(i\\) gets their own hat. Then total number with their own hat is \\(X = \sum_i X_i\\), and

\\[ E[X] = \sum_{i=1}^n E[X_i] = \sum_i P(X_i = 1) = \sum_i \frac{1}{n} = 1 \\]

Each person gets their own hat with probability \\(1/n\\) by symmetry. By linearity, the expected total is exactly 1, regardless of \\(n\\), and regardless of the fact that "person 1 got their own hat" and "person 2 got their own hat" are *not* independent events.

The expected number does not even depend on \\(n\\). That is striking enough on its own.

## Coupon collector

You buy cereal boxes; each contains a uniformly random one of \\(n\\) coupon types. What is the expected number of boxes you must buy to collect all \\(n\\)?

Let \\(T_k\\) be the number of additional boxes needed to collect a new coupon, given you already have \\(k - 1\\) of the types. Each box independently shows a new coupon with probability \\((n - k + 1)/n\\), so \\(T_k\\) is geometric and \\(E[T_k] = n / (n - k + 1)\\).

Total time \\(T = T_1 + T_2 + \cdots + T_n\\). Linearity:

\\[ E[T] = \sum_{k=1}^{n} \frac{n}{n - k + 1} = n \sum_{j=1}^{n} \frac{1}{j} = n H_n \\]

where \\(H_n\\) is the \\(n\\)-th harmonic number, approximately \\(\ln n + \gamma\\). So expected number of cereal boxes to collect 200 coupons is about \\(200 \ln 200 \approx 1060\\), not 200.

Notice the \\(T_k\\) are *not* independent of the order — the random sequence of coupons couples them — but linearity does not care.

## Random graphs: triangles in G(n, p)

In the Erdős–Rényi random graph \\(G(n, p)\\), each of the \\(\binom{n}{2}\\) potential edges is present independently with probability \\(p\\). What is the expected number of triangles?

Let \\(X_T\\) be the indicator that a given triple of vertices \\(T\\) forms a triangle (all three edges present). By independence of edges, \\(E[X_T] = p^3\\). The total number of triangles is \\(X = \sum_T X_T\\) over all \\(\binom{n}{3}\\) triples, so

\\[ E[X] = \binom{n}{3} p^3 \\]

The triples *overlap* — different triples share edges, so the indicators are not independent — but linearity ignores this. Want to know if your random graph likely has at least one triangle? Plug in numbers and read off when \\(E[X]\\) crosses 1.

## Karger's min-cut algorithm

Karger's randomized min-cut algorithm contracts random edges until two vertices remain. The probability it returns the actual minimum cut is at least \\(1/\binom{n}{2}\\). Run the algorithm \\(\binom{n}{2} \log n\\) times and take the smallest cut found, you get the right answer with high probability. The analysis hinges on linearity-of-expectation arguments to count edges in cuts during contractions.

## Probabilistic-method proof of existence

You want to prove a graph with property \\(P\\) exists. Define a random graph in some natural way; let \\(X\\) be the number of structures violating \\(P\\); compute \\(E[X]\\). If \\(E[X] < 1\\), some realization of the random graph has \\(X = 0\\), which is a graph with the desired property.

Example: a graph on \\(n\\) vertices with no clique or independent set of size \\(2 \log_2 n\\) exists. Define a uniformly random graph on \\(n\\) vertices. The expected number of cliques *or* independent sets of size \\(k\\) is \\(\binom{n}{k} 2^{1-\binom{k}{2}}\\). For \\(k > 2 \log_2 n\\), this is less than 1 for large \\(n\\). So such a graph exists. Linearity inside; existence outside.

## Why this is wonder, not just a trick

The asymmetry between expectation (linear, free) and variance (nonlinear, requires independence to add) is the secret of the subject. A *count* is just a sum of indicators, and the expectation of a count is the sum of probabilities of each thing being counted, regardless of whether those things interact. That fact dissolves what would have been a horrible inclusion-exclusion into a one-line probability argument.

It also generalizes: the integral of a sum is the sum of integrals; the trace of a sum of matrices is the sum of their traces; the dimension of a direct sum of vector spaces is the sum of the dimensions. Linearity is everywhere. Probability inherits it from the linearity of integration, and counting inherits it from probability through the indicator-function trick. Once you start looking for it, you see it.

## Where to go deeper

- Mitzenmacher and Upfal, *Probability and Computing*, Chapters 2–3. Worked examples building from coin flips to graph algorithms.
- Alon and Spencer, *The Probabilistic Method*. The whole book is "linearity of expectation, variance, and second moment, applied to existence proofs." If you want to see this technique used at full power, this is the reference.
