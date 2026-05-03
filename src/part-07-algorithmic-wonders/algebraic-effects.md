# Algebraic effects and handlers

Side effects in a program — printing, raising exceptions, mutating state, reading from a database, asking the user a question — are usually language primitives. You either have them or you do not. With *algebraic effects*, they become user-defined. Any kind of effect can be declared as an operation, and any handler around the code can intercept and reinterpret what that operation means.

The result is structured non-local control flow that subsumes exceptions, generators, async/await, dependency injection, and several other features that languages traditionally bake in. Handlers compose. The same code can run with different effect handlers in different contexts, with no code changes.

The construction was formalized by Plotkin and Power in 2003 and is now a first-class feature in OCaml 5, Effekt, Koka, Eff, and several other languages. It is the natural successor to monads as the abstraction for "effects in pure code."

## Declaring an operation

```ocaml
type _ Effect.t += Ask : string -> string Effect.t
```

This declares an effect operation `Ask` of type "string in, string out." It does *not* implement the effect. A program can call `perform (Ask "name")` and a handler around the call site decides what happens.

## A handler

```ocaml
let with_default_input default body =
  try body () with
  | effect (Ask _prompt), k -> continue k default
```

This handler intercepts every `Ask` operation in `body ()` and resumes with `default`. The body might do something arbitrary; whenever it calls `Ask`, this handler responds with `default`.

```ocaml
let prog () =
  let name = perform (Ask "name") in
  let city = perform (Ask "city") in
  Printf.sprintf "%s from %s" name city

with_default_input "anonymous" prog
(* Returns: "anonymous from anonymous" *)
```

A different handler:

```ocaml
let with_console body =
  try body () with
  | effect (Ask prompt), k ->
      print_string prompt; print_string ": ";
      let line = read_line () in
      continue k line

with_console prog
(* Asks the user, then returns "Alice from Berlin" *)
```

Same `prog`, two different handlers, two different behaviors. The handler is a layer of interpretation around the effect-using code.

## Resumption: the magic

The handler receives `k`, the *continuation* of the effect. `continue k value` resumes the program at the point of the `perform`, supplying `value` as the result. The handler can also choose *not* to resume, abandoning the program; or to resume *more than once*, replaying the program with different values.

A simple example: handler that runs the body twice with different values, returning a pair.

```ocaml
let with_two_choices a b body =
  match body () with
  | x -> (x, x)
  | effect (Ask _), k -> (continue (Obj.copy k) a, continue k b)
```

Now the program runs twice, once with `a`, once with `b`, returning a pair of results. This is *backtracking* via captured continuations.

## Why this subsumes other constructs

**Exceptions**: just an effect with no resumption. `raise e` performs an effect; the handler matches it and *does not call `continue`*, so the program terminates.

**Generators / yield**: `yield value` performs an effect; the handler captures `k` and returns it as a "next" function. The caller invokes `k` when ready, resuming the generator until the next `yield`.

**Async/await**: `await promise` performs an effect; the handler suspends `k` until the promise resolves, then resumes with the resolved value.

**Mutable state**: `get` and `set` operations; handler maintains the state and threads it through resumptions.

**Reader monad / dependency injection**: a `Get` effect that returns a value from environment; handler supplies the value.

**Probabilistic programming**: `sample` effect; handler implements the inference algorithm (rejection sampling, MCMC, etc.).

**Logging / tracing**: `Log` effect; handler decides whether to print, file-log, accumulate, or discard.

These are all *the same construct* with different declarations and different handlers. In a language with algebraic effects, you do not need to add new primitives for any of these. You define the effect once, write handlers that implement it, and the language's existing machinery does the rest.

## How is this different from monads

Monadic programming requires every effectful function to be marked with the monad in its type, and combining different effects requires monad transformers (or effect-row variants in Haskell). Code becomes monad-coloured, and combining libraries with different monads is a constant friction.

Algebraic effects let effectful code look like ordinary direct-style code. The effect is in the type (in some languages with effect rows), but the syntax is just a function call. Handlers compose by nesting: an outer handler sees only the effects that inner handlers did not catch. There is no transformer pyramid; there is no need to redefine the function for each effect set.

In typed languages with effect rows (Eff, Koka, Effekt, the experimental Helium), the type checker tracks which effects each expression performs. Functions that perform `Log` and `Ask` have type signatures that say so. Handlers reduce the effect set, and a fully-handled program has no effects in its type.

## Implementation

In OCaml 5, effect handlers are implemented via lightweight, one-shot continuations on a special segmented stack. Performing an effect captures the current stack segment as a delimited continuation, hands it to the handler. `continue k` resumes by jumping back to that captured stack. Implementations of multi-shot continuations (those allowing `continue` to be called more than once) require copying the stack, which is more expensive.

Modern compilers can sometimes generate handler code that is no slower than direct exception handling — the abstractions compose down to the same machine code as hand-written control flow.

## Why this is different from CPS

CPS makes continuations explicit *in the source code*. Algebraic effects make continuations explicit *only inside handlers*. The user code looks direct-style — no callbacks, no continuation parameters, no `then`/`bind` chains. The handler is the only place that sees the explicit control flow.

This means algebraic effects give you the expressive power of CPS (you can implement anything CPS can) with the syntactic ergonomics of direct-style code. You write your algorithm normally, and the handler decides what the side effects mean.

## Effects vs monads vs callbacks

In a language without algebraic effects, async code with the syntactic shape of synchronous code requires either:

- **Callbacks**: explicit, painful (callback hell, error propagation by hand).
- **Monads (Promise / Future)**: requires every async function to return a different type, and `.then` chains spread through the codebase.
- **Async/await language feature**: the language adds direct support, with a runtime to manage suspended continuations.

Algebraic effects subsume all of these. The language's effect handlers can implement async/await as a library, exceptions as a library, generators as a library. You do not need each one to be a language feature.

This is the pitch for algebraic effects in language design: they are the *uniform* mechanism that subsumes a dozen ad-hoc control-flow features.

## Where they show up in practice

- **Eff**, **Koka**, **Multicore OCaml** (now OCaml 5): research and production languages with first-class effect handlers.
- **React's Suspense and Server Components**: not literal algebraic effects, but the rendering engine treats `useSomething` calls as effect operations and `<Suspense>` as a handler.
- **Dan Abramov's "Algebraic Effects for the Rest of Us"**: explanation for the React community.
- **F#'s Computation Expressions**, **Scala's Effekt library**, **OCaml's effect handlers**: production deployments.

## The wonder

You can write a program in direct style — no callbacks, no monad transformers, no nested then-chains — and have its semantics determined by handlers placed around it. The same code runs synchronously, asynchronously, with mocked I/O, with deterministic randomness, with state-tracking, with backtracking, with nondeterminism — all by changing the handler, not the code.

The wonder is that the technical machinery underlying this is *delimited continuations*. The same primitive that gives you `call/cc` (in Scheme) and the suspended state of a coroutine is, when packaged as effects-and-handlers, the unifying construct that lets you implement every nonlocal-control-flow feature as a library. Every language that has built `async/await`, exceptions, generators, and dependency injection as separate features could, instead, expose effect handlers and let users build any of them — plus features no one has thought of yet — with no language change.

## Where to go deeper

- Plotkin and Pretnar, *Handlers of Algebraic Effects*, ESOP 2009. The defining paper.
- Pretnar, *An Introduction to Algebraic Effects and Handlers*, MFPS 2015. Lecture-note-level introduction.
- The Multicore OCaml documentation and *Programming with Effect Handlers in OCaml 5* (Sivaramakrishnan et al., 2021). Modern engineering.
