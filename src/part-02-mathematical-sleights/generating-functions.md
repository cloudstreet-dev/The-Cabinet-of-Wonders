# Generating functions

You can prove a fact about an infinite sequence of integers by treating the sequence as the coefficients of a power series, doing algebra with that series as if it were a single mathematical object, and reading the answer off the resulting series. The integers do not know they are coefficients. The power series does not converge anywhere meaningful. The technique works anyway.

It is one of the most consistently surprising tools in combinatorics, because it transforms questions about counting — discrete, finite, often messy — into questions about formal algebra, where you have a century of techniques and they all apply.

## A sequence becomes a function

Given any sequence \\(a_0, a_1, a_2, \dots\\), define its ordinary generating function:

\\[ A(x) = \sum_{n=0}^{\infty} a_n x^n = a_0 + a_1 x + a_2 x^2 + \cdots \\]

The variable \\(x\\) is a formal symbol. The series is "formal": we do not ask whether it converges. We ask only that it follow the algebraic rules of power series. If \\(A(x)\\) and \\(B(x)\\) are two such series, then

\\[ A(x) + B(x) = \sum_n (a_n + b_n) x^n \\]
\\[ A(x) \cdot B(x) = \sum_n \left(\sum_{k=0}^n a_k b_{n-k}\right) x^n \\]

That last identity is the crucial one. The coefficient of \\(x^n\\) in the product is the *convolution* of the two sequences. Convolutions show up everywhere in counting — and now they are just multiplications.

## The Fibonacci move

Define Fibonacci numbers by \\(F_0 = 0\\), \\(F_1 = 1\\), \\(F_{n+2} = F_{n+1} + F_n\\). What is a closed form?

Define \\(F(x) = \sum_n F_n x^n\\). The recurrence translates directly to an identity on \\(F(x)\\):

\\[ \sum_{n \geq 0} F_{n+2} x^{n+2} = \sum_{n \geq 0} F_{n+1} x^{n+2} + \sum_{n \geq 0} F_n x^{n+2} \\]

\\[ F(x) - F_0 - F_1 x = x(F(x) - F_0) + x^2 F(x) \\]

\\[ F(x) - x = x F(x) + x^2 F(x) \\]

\\[ F(x) (1 - x - x^2) = x \\]

\\[ F(x) = \frac{x}{1 - x - x^2} \\]

That is the generating function for Fibonacci. Now factor the denominator. Let \\(\phi = \frac{1 + \sqrt{5}}{2}\\) and \\(\psi = \frac{1 - \sqrt{5}}{2}\\). Then \\(1 - x - x^2 = (1 - \phi x)(1 - \psi x)\\), and partial fractions give

\\[ F(x) = \frac{1}{\sqrt{5}}\left( \frac{1}{1 - \phi x} - \frac{1}{1 - \psi x} \right) \\]

\\[ = \frac{1}{\sqrt{5}} \sum_n (\phi^n - \psi^n) x^n \\]

Reading off the coefficient of \\(x^n\\):

\\[ F_n = \frac{\phi^n - \psi^n}{\sqrt{5}} \\]

This is Binet's formula, derived in seven lines of formal algebra. Notice what we did *not* do: induction, characteristic equations, careful case analysis. The recurrence got translated into a polynomial identity, the polynomial got factored, and the partial-fraction expansion gave the answer.

## Combinations as products

The sleight is that products of generating functions are convolutions of sequences, and convolutions of sequences are exactly what you compute when you split objects into independent parts.

How many ways can you make change for \\(n\\) cents using pennies, nickels, dimes, quarters? The answer is the coefficient of \\(x^n\\) in

\\[ \frac{1}{(1 - x)(1 - x^5)(1 - x^{10})(1 - x^{25})} \\]

The first factor expands to \\(1 + x + x^2 + \cdots\\) — choose any number of pennies. The second is \\(1 + x^5 + x^{10} + \cdots\\) — any number of nickels (each contributing 5 cents). And so on. Multiplying these convolves the choices. The coefficient of \\(x^n\\) counts ordered tuples of (penny count, nickel count, dime count, quarter count) summing to \\(n\\). Answer: extract the coefficient.

There is no clever combinatorial argument needed. The mechanism — choose a count from each denomination, sum to \\(n\\) — is exactly what generating-function multiplication does.

## Counting binary trees

Let \\(C_n\\) be the number of binary trees on \\(n\\) nodes. A binary tree is either empty (1 way) or a root with a left and right subtree. So

\\[ C_n = \sum_{k=0}^{n-1} C_k \, C_{n-1-k} \quad (n \geq 1), \quad C_0 = 1 \\]

Define \\(C(x) = \sum_n C_n x^n\\). The recurrence — sum-of-products of subsequences — is a convolution. Convolution is multiplication in the generating-function world:

\\[ \sum_{n \geq 1} C_n x^n = x \sum_{n \geq 1} \sum_{k=0}^{n-1} C_k C_{n-1-k} x^{n-1} = x \cdot C(x)^2 \\]

So \\(C(x) - 1 = x C(x)^2\\). Solve the quadratic in \\(C\\):

\\[ x C^2 - C + 1 = 0 \quad \implies \quad C(x) = \frac{1 - \sqrt{1 - 4x}}{2x} \\]

(The other root has the wrong constant term.) Expanding \\(\sqrt{1 - 4x}\\) by the binomial series and reading off coefficients gives

\\[ C_n = \frac{1}{n+1} \binom{2n}{n} \\]

The Catalan numbers. Three lines of algebra; an answer that took the original combinatorialists an entirely different argument to find.

## The exponential variant

For sequences where the natural operation is "labelled" rather than "unlabelled" — permutations, structures on labelled vertices — use exponential generating functions:

\\[ \hat{A}(x) = \sum_n a_n \frac{x^n}{n!} \\]

Now multiplication of EGFs corresponds to a different convolution: \\(\hat{A}(x) \hat{B}(x) = \sum_n \left( \sum_k \binom{n}{k} a_k b_{n-k} \right) \frac{x^n}{n!}\\). The factor \\(\binom{n}{k}\\) inside is the choice of which \\(k\\) labels go to the \\(A\\)-part. EGFs are the natural language for labeled combinatorics.

The exponential function \\(e^x = \sum_n \frac{x^n}{n!}\\) is the EGF of the constant sequence \\(1, 1, 1, \dots\\). Its meaning: there is one way to put a labeled "trivial structure" on every set of size \\(n\\). Then \\(e^{f(x)}\\) is the EGF of "sets of \\(f\\)-structures" — and this gives, in two symbols, the EGF of partitions of a set, or the EGF of permutations decomposed into cycles, depending on what \\(f\\) is.

## The wonder

The integers \\(a_0, a_1, a_2, \dots\\) live in a discrete world. The function \\(A(x) = \sum a_n x^n\\) lives in a continuous one. The two are not, in any geometric sense, the same object. But every theorem you can prove about \\(A(x)\\) using analysis — partial fractions, formal differentiation, root extraction, composition with other series — translates back into a theorem about the original sequence, because the coefficient extraction operator \\([x^n]\\) is just bookkeeping.

So you trade combinatorial reasoning for algebraic reasoning. The trade is enormously profitable: there is a vast and well-developed theory of formal series, and almost none of it had to be developed for combinatorics. It just turned out that counting things and shuffling formal series are the same activity in different uniforms.

## Where to go deeper

- Wilf, *generatingfunctionology* (3rd ed., free online). The standard introduction. Worked examples carry the whole subject.
- Flajolet and Sedgewick, *Analytic Combinatorics* (free online). The mature theory: turns generating functions into asymptotic-counting machines via singularity analysis.
