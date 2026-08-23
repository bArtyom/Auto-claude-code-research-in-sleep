# Round 4 Pipeline Summary — SemLibSQL Pilot

**Date:** 2026-08-23
**Workflow:** research-lit → novelty-check → research-review (internal/provisional) → research-refine → experiment-plan
**Status:** READY FOR GATE-A PILOT DESIGN; formal cross-model acceptance unavailable.

## 1. What changed in this round

The largest new novelty threat is **ReGAL (ICML 2024)**, which already learns execution-validated reusable abstractions from program corpora. Combined with DreamCoder, Stitch, babble, SQL equivalence provers, and Text2SQL semantic-memory work, this rules out a broad "learn reusable SQL functions from history" contribution.

SemLibSQL is therefore narrowed further.

## 2. Final narrow thesis

> Enterprise SQL workloads contain repeated business semantics implemented through heterogeneous relational programs. A conservative SQL/warehouse-semantic canonicalizer can expose these latent equivalence families and enable automatic induction of typed operators that improve unseen motif composition beyond generic program-library learning and structured verified-query memory.

## 3. Dominant novelty test

Before any full agent implementation, answer one question:

> **Does semantic canonicalization find useful cross-realization abstractions that AST/Stitch/ReGAL-style methods miss?**

If no, kill the project.

## 4. Round-4 deliverables

- `LITERATURE_DEEPENING.md` — adjacent literature and the ReGAL collision.
- `NOVELTY_RECHECK.md` — narrowed claims and reviewer-attack simulation.
- `PILOT_DATASET_PROTOCOL.md` — corpus, motif, hard-positive/negative, verification, and composition split design.
- `CANONICALIZATION_DESIGN.md` — conservative three-state equivalence architecture.
- `CLAIMS_AND_KILL_GATES.md` — pre-registered claims and stop conditions.
- `PILOT_RUNBOOK.md` — cheapest-first execution order.

## 5. Required competitors

The minimum serious baseline set is now:

1. normalized SQL templates;
2. AST anti-unification;
3. Stitch-like library learning;
4. ReGAL-style execution-validated refactoring;
5. structured verified semantic memory;
6. SQL formal-equivalence assistance;
7. human/oracle semantic library.

## 6. Gate A

Use 200–500 verified programs over 8–12 motifs and 3–4 schemas/projects.

Construct hard positives where one motif has very different realizations, plus hard negatives where similar syntax has different semantics.

PASS only if semantics-aware normalization materially improves useful abstraction discovery at controlled false-merge rate.

## 7. Gate B

Only after Gate A passes, test held-out compositions where every primitive was seen in history but the complete composition was not.

Primary comparison:

> SemLibSQL vs strongest structured-memory / generic-library baseline under matched context.

Continue investment only for a robust accuracy gain or a substantial accuracy-preserving efficiency gain.

## 8. Explicitly rejected complexity

For now, do not add:

- RL;
- multi-agent orchestration;
- continual learning;
- automatic semantic-view materialization;
- long-horizon memory versioning;
- complex learned routers;
- model fine-tuning.

Those are downstream options only if the abstraction mechanism survives falsification.

## 9. Research-review status

The current session does not expose the independent reviewer backend required by the repository's formal review contract. Consequently:

```yaml
review_independence: same-family/internal
formal_acceptance: unavailable
status: provisional
```

No cross-model acceptance claim is made.

## 10. Next research action

The next non-implementation step is to turn the motif list into a **concrete pilot corpus manifest** with 30–50 seed examples and an annotation/equivalence rubric. After that artifact is reviewed, implementation of corpus-generation/canonicalization tooling can begin under the development workflow.