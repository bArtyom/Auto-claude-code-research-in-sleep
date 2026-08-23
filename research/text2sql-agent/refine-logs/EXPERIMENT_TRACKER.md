# Experiment Tracker — SemLibSQL-Γ

**Created:** 2026-08-23  
**State:** **NOT STARTED — idea stage intentionally stopped before empirical work**

| ID | Block | Purpose | Status | Result | Decision |
|---|---|---|---|---|---|
| A0 | Corpus instantiation | executable hard positives/negatives | NOT STARTED | — | — |
| A1 | Token/AST baselines | establish syntax floor | NOT STARTED | — | — |
| A2 | Stitch-style library | generic syntax abstraction | NOT STARTED | — | — |
| A3 | babble/LLMT fixed theory | strongest theory-aware generic baseline | NOT STARTED | — | — |
| A4 | ReGAL-style refactoring | execution-validated generic abstraction | NOT STARTED | — | — |
| A5 | SemLibSQL global | isolate contextual contracts | NOT STARTED | — | — |
| A6 | SemLibSQL-Γ | main Gate-A method | NOT STARTED | — | PASS/FAIL pending |
| B0–B7 | Held-out composition | library vs memory | BLOCKED ON GATE A | — | — |
| C | Scope violation audit | negative-transfer safety | BLOCKED ON GATE A/B | — | — |

## Gate A

**Question:** Does warehouse-conditioned equivalence change the abstraction precision–recall/safety frontier beyond strong generic library learning?

**Status:** NOT RUN.

## Gate B

**Question:** Do induced scoped operators improve unseen motif compositions beyond strong verified memory?

**Status:** BLOCKED until Gate A passes.

## Gate C

**Question:** Do applicability contracts prevent negative transfer under deliberately violated warehouse assumptions?

**Status:** BLOCKED until the learned library is useful enough to test.

## Notes

- Do not enter results retrospectively without preserving raw artifacts and exact configuration.
- Do not proceed to B/C after a failed A unless the research question is explicitly changed and re-reviewed.
- A failed Gate A terminates the current SemLibSQL mechanism; it should not trigger automatic addition of RL, search, multi-agent review, or extra memory components.
