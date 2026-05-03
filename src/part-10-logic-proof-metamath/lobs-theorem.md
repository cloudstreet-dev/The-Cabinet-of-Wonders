# Löb's theorem

A formal system that can prove "if I can prove \\(P\\), then \\(P\\) is true" (for some specific \\(P\\)) can already prove \\(P\\). The mere fact that the system has *internalized its own soundness* for \\(P\\) lets it actually prove \\(P\\). So either \\(P\\) was already provable, or the system cannot prove its own soundness for \\(P\\).

Löb's theorem (1955) sounds like a curiosity. It is the cleanest statement about the limits of self-reference in formal systems, and it has consequences that ripple through theoretical computer science, modal logic, and even AI safety reasoning.

## The statement

Let \\(\text{PA}\\) be Peano Arithmetic (or any sufficiently strong formal theory containing arithmetic), and let \\(\text{Provable}(\ulcorner \cdot \urcorner)\\) be the predicate "is provable in \\(\text{PA}\\)" (encoded as a formula via Gödel numbering — see *Gödel's coding trick*).

**Löb's theorem.** For any sentence \\(P\\):

\\[ \text{PA} \vdash \text{Provable}(\ulcorner P \urcorner) \to P \quad \implies \quad \text{PA} \vdash P \\]

If you can prove "if \\(P\\) is provable, then \\(P\\)," you can already prove \\(P\\).

## The proof

Sketch (using Gödel's diagonal lemma and the Hilbert-Bernays-Löb derivability conditions):

1. Suppose \\(\text{PA} \vdash \text{Provable}(\ulcorner P \urcorner) \to P\\).
2. By the diagonal lemma, there is a sentence \\(L\\) such that
\\[ L \leftrightarrow (\text{Provable}(\ulcorner L \urcorner) \to P) \\]
3. From the fact that \\(L\\) is provably equivalent to \\(\text{Provable}(\ulcorner L \urcorner) \to P\\), Hilbert-Bernays-Löb derivability conditions give:
\\[ \text{PA} \vdash \text{Provable}(\ulcorner L \urcorner) \to \text{Provable}(\ulcorner \text{Provable}(\ulcorner L \urcorner) \to P \urcorner) \\]
4. By another HBL condition (modal axiom K, which formalizes modus ponens inside provability):
\\[ \text{PA} \vdash \text{Provable}(\ulcorner L \urcorner) \to (\text{Provable}(\ulcorner \text{Provable}(\ulcorner L \urcorner) \urcorner) \to \text{Provable}(\ulcorner P \urcorner)) \\]
5. Use the second HBL condition (\\(\text{Provable}(P) \to \text{Provable}(\text{Provable}(P))\\)) to simplify to:
\\[ \text{PA} \vdash \text{Provable}(\ulcorner L \urcorner) \to \text{Provable}(\ulcorner P \urcorner) \\]
6. By assumption (step 1), \\(\text{Provable}(\ulcorner P \urcorner) \to P\\). Chain:
\\[ \text{PA} \vdash \text{Provable}(\ulcorner L \urcorner) \to P \\]
7. By the equivalence in step 2, this means \\(\text{PA} \vdash L\\).
8. From \\(L\\) being a theorem and the first HBL condition, \\(\text{PA} \vdash \text{Provable}(\ulcorner L \urcorner)\\).
9. Combining steps 6 and 8: \\(\text{PA} \vdash P\\). \\(\blacksquare\\)

The proof is short but each step uses a derivability condition that itself takes work to establish. The whole thing is, in essence, a self-referential diagonal argument applied to a sentence \\(L\\) that says "if I am provable, then \\(P\\)."

## Gödel's incompleteness as a corollary

Löb's theorem implies Gödel's second incompleteness theorem:

If \\(\text{PA}\\) is consistent, set \\(P = \bot\\) (a contradiction). The hypothesis of Löb's theorem becomes \\(\text{PA} \vdash \text{Provable}(\ulcorner \bot \urcorner) \to \bot\\), which is just "PA proves its own consistency" (Provable(false) is false). The conclusion would be \\(\text{PA} \vdash \bot\\), contradicting consistency. So PA cannot prove its own consistency. Gödel 2.

So Löb's theorem strictly generalizes Gödel's second theorem: if PA could prove the soundness of provability for *any* statement (i.e., "if \\(P\\) is provable then \\(P\\)"), it would already have proved \\(P\\), and as a special case, it cannot consistently prove that "if \\(\bot\\) is provable then \\(\bot\\)" (i.e., its own consistency).

## What this rules out

You might naively think a formal system could be designed to be its own meta-theory: prove its own consistency, prove the soundness of its own proofs, etc. Löb's theorem makes this impossible (under modest assumptions). Any formal system strong enough to do arithmetic and arithmetize its own provability either *already proves \\(P\\)* or *cannot prove that proving \\(P\\) implies \\(P\\)*. In particular, it cannot prove *non-trivial soundness statements* about itself.

A formal system can have a *truth predicate* for arithmetic only externally — there is no internal definition by Tarski's undefinability. Similarly, soundness of provability cannot be a theorem of the system itself for non-provable statements.

## Modal logic GL

Provability behaves like a modal operator: \\(\Box P\\) := "\\(P\\) is provable." The HBL derivability conditions match the axioms of the modal logic GL (Gödel-Löb logic):

- K: \\(\Box(P \to Q) \to (\Box P \to \Box Q)\\)
- 4: \\(\Box P \to \Box \Box P\\)
- L (Löb's axiom): \\(\Box(\Box P \to P) \to \Box P\\)

GL is decidable, has a clean Kripke semantics (with finite, transitive, irreflexive frames), and characterizes the provability fragment of arithmetic exactly (Solovay's completeness theorem 1976). Löb's theorem is, in this sense, the modal axiom that captures "self-referring soundness collapses to provability."

## Curious AI / agency consequences

Löb's theorem has been invoked in discussions of self-improving AI agents:

If a rational agent reasons about itself in a formal system and tries to prove "if I prove that action \\(A\\) is good, then \\(A\\) is good," Löb's theorem says: either it can prove \\(A\\) is good directly, or it cannot prove the soundness statement. In particular, designing an agent that trusts its own future judgments without question runs into Löb-style limits.

This is a small but real strand of work in AI safety (the "Löbian obstacle" to recursive self-reflection). The technical impact is limited; the conceptual point — that self-trust is bounded by what you can already establish without self-trust — is real.

## Why this is wonder

You would expect that a system as expressive as arithmetic, capable of formalizing its own provability and proving many things, could *also* prove its own soundness for individual statements. "If I prove \\(P\\), then \\(P\\) is true" seems like a plausible meta-theoretical claim. But it cannot be a theorem of the system itself, except in the trivial case when \\(P\\) was already provable.

The wonder is the precision. The constraint is not "you cannot prove your own consistency" — that would be Gödel 2, a special case. The constraint is "you cannot prove your own *soundness for any specific statement*" — for any \\(P\\) you have not already proved, the meta-statement is itself unprovable.

In some sense, this is the deepest limit of formal self-reflection. A system can talk about its own provability; it can prove modus ponens about itself; it can chain inferences about its own theorems. But it cannot leverage this metamachinery to *increase its own deductive power* by proving soundness statements as shortcuts. The internalized self-knowledge does not help.

## Where to go deeper

- Löb, *Solution of a Problem of Leon Henkin*, JSL 1955. The original.
- Boolos, *The Logic of Provability* (Cambridge, 1993). Modern treatment of GL and provability logic.
