# CPS transformation

Take any program. Mechanically transform it so that no function ever returns. Instead, every function takes an extra argument — its *continuation*, which is the function representing "what to do with my result." After the transformation, the program is a chain of tail calls that flow forward forever. The original return paths are gone; control flow is now a continuation-passing river.

This is the CPS (continuation-passing style) transformation. It looks like a syntactic curiosity at first. It turns out to be the foundation of compiler intermediate representations, a way to implement exceptions and generators and async/await without language support, and the secret behind why functional and imperative programming feel different even though they compute the same things.

## The transformation

Direct-style code:

```scheme
(define (square x) (* x x))
(define (sum-of-squares a b) (+ (square a) (square b)))
(sum-of-squares 3 4)
```

CPS-transformed:

```scheme
(define (square/k x k) (k (* x x)))

(define (sum-of-squares/k a b k)
  (square/k a (lambda (a^2)
    (square/k b (lambda (b^2)
      (k (+ a^2 b^2)))))))

(sum-of-squares/k 3 4 (lambda (result)
  (display result)))
```

Each function gained a `k` (continuation) parameter. Instead of returning, it calls `k` with the result. Composing functions becomes nesting continuations. The "value flow" of the original is now an explicit "what happens next" parameter.

The transformation is mechanical. There is a Plotkin-style algorithm that takes any direct-style expression and produces its CPS form. The transformation is:

\\[ [\![ x ]\!]_k = k \, x \\]
\\[ [\![ \lambda x. e ]\!]_k = k \, (\lambda x. \lambda k'. \, [\![ e ]\!]_{k'}) \\]
\\[ [\![ e_1 \, e_2 ]\!]_k = [\![ e_1 ]\!]_{\lambda f. \, [\![ e_2 ]\!]_{\lambda v. \, f \, v \, k}} \\]

That is the whole conversion, in three rules. It is purely syntactic; no runtime trickery.

## Why no function returns

In CPS, the *only* operation is "call the continuation with my result." There is no return statement. The return address is now the continuation parameter, passed explicitly.

A clean way to think of it: in machine-code terms, every function call in CPS is a *jump*, never a *call/return*. The stack is irrelevant; "what happens next" is in the continuation, and the continuation is a value. There is no implicit control-flow pointer.

This has a lovely consequence: every CPS program is in *tail-call form*. Every call is the last thing the calling function does. So a runtime that supports tail-call optimization can run CPS programs without consuming stack space, no matter how deep the recursion.

## Why compilers love it

Compilers transform code into intermediate representations to optimize it. CPS is one of the standard intermediate representations because:

- **Every control-flow construct becomes uniform.** Conditionals, loops, exceptions, function calls — all become continuations being applied. No special cases.
- **Tail calls are explicit and easy to optimize.** "Replace the call with a jump" works directly on CPS code.
- **Optimizations compose.** Beta reduction, dead-code elimination, common-subexpression elimination all have clean rules in CPS.
- **Direct generation of machine code.** CPS is close to register-transfer form. Each continuation can become a basic block; each call becomes a branch.

The Glasgow Haskell Compiler (GHC) uses a CPS-like form (called Spineless Tagless G-machine, then later "Cmm"). The OCaml compiler, the Scheme R6RS standard, the LLVM IR (in some passes) — all use CPS or CPS-like forms internally. Andrew Appel's *Compiling with Continuations* (1992) made this approach mainstream.

## Implementing exceptions in pure CPS

In direct style, exceptions need special language support. In CPS, they fall out for free:

Each function takes *two* continuations: \\(k\\) for "normal return" and \\(k_e\\) for "raise exception." When everything goes well, call \\(k\\). When something fails, call \\(k_e\\).

```scheme
(define (sqrt/cps x k k_e)
  (if (< x 0)
      (k_e "negative")
      (k (sqrt-positive x))))
```

A `try ... catch` is just installing a custom \\(k_e\\). A `raise` is calling the current \\(k_e\\). Stack unwinding happens because \\(k_e\\) was captured at the `try`, and calling it skips back to that level of the continuation tree.

No language support needed. CPS makes this a programming idiom.

## Implementing generators and coroutines

A generator function — `yield` returning to the caller, then resuming where it left off — is just a captured continuation. Each `yield` produces a value and the *continuation* that represents "where to resume." The caller invokes that continuation when it wants the next value.

```scheme
(define (range/gen start end k)
  (if (= start end)
      (k 'done)
      (k start (lambda () (range/gen (+ start 1) end k)))))
```

`range/gen` calls its continuation with the current value and a thunk to get the next. The caller can choose when to resume.

Coroutines are similar: continuations as named entry points; switching is calling the saved continuation. The scheduler is just a loop calling continuations.

This is exactly how async/await is implemented under the hood in Python, JavaScript, C#, and Rust: each `await` is a continuation; the runtime captures the surrounding "what to do next" and registers it as a callback for when the awaited operation completes.

## First-class continuations

Some languages give you continuations as first-class values: Scheme's `call/cc` (call-with-current-continuation) captures the current continuation and binds it to a variable. You can then invoke it later, transferring control back to that point — even after the surrounding function has returned.

```scheme
(+ 1 (call/cc (lambda (k) (+ 2 (k 3)))))
```

When `(k 3)` is invoked, it discards the surrounding `(+ 2 ...)` and jumps directly back to the outer context, returning 3. The result is `(+ 1 3) = 4`, *not* `6`. The captured continuation `k` represented "add 1 and return"; calling it short-circuits the inner computation.

`call/cc` is wildly powerful and wildly confusing. It can implement exceptions, generators, multitasking, backtracking — anything that requires nonlocal control flow. It is also infamous for producing programs that are hard to reason about.

## The wonder

The CPS transformation is a syntactic operation that takes any program and produces an equivalent program with no return statements. The transformation is fully mechanical. Yet the transformed program has a different *shape*: control flow is data; what happens next is a value you can manipulate, capture, copy, and re-invoke.

That equivalence — that you can *freely transform* between direct style (where control is implicit) and CPS (where it is explicit) — is why the inside of a compiler can rearrange your code in nontrivial ways. It is why exceptions, generators, async/await, and coroutines can all be implemented in a language that only supports function calls. It is why functional and imperative programming, despite the surface differences, sit on the same theoretical foundation.

The wonder is in the equivalence. Returning is not a primitive. It is a convention. Underneath every return is a "where to go next," and the CPS transformation just makes that "where" explicit. Once you have seen this, you can never unsee it. Every return in your code is silently passing a continuation that the language is hiding from you.

## Where to go deeper

- Plotkin, *Call-by-name, call-by-value, and the lambda calculus*, Theoretical Computer Science 1975. The original CPS transformation, with semantic correctness.
- Appel, *Compiling with Continuations* (Cambridge, 1992). The book on CPS as a compiler IR.
- Friedman and Wand, *Essentials of Programming Languages*, Chapter 5. Modern textbook treatment.
