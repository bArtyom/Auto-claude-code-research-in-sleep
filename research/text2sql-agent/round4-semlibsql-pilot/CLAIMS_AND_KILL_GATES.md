# SemLibSQL Claims Matrix and Kill Gates

> Workflow role: adversarial review / experiment pre-registration
> Purpose: prevent post-hoc story inflation.

## 1. Dominant claim

### C1 — Semantic canonicalization enables abstraction beyond generic library learning

A SQL/warehouse-semantic representation should recover reusable motif families across heterogeneous SQL realizations that syntactic or generic execution-refactoring library learners fail to unify safely.

**Evidence required**

- hard-positive recall gain over AST/Stitch/ReGAL-style baselines;
- low false-merge rate on hard negatives;
- examples spanning distinct SQL realization families;
- ablation showing the gain comes from semantic layers rather than trivial normalization.

**Kill C1 if**

- semantic method is within noise of strong syntactic/refactoring baseline;
- gains are only alias/formatting/CTE normalization;
- false merges rise materially;
- benefit disappears after giving baseline safe SQL rewrite normalization.

---

## 2. Main downstream claim

### C2 — Learned library beats structured verified memory on unseen motif composition

**Evidence required**

- held-out composition split;
- strongest verified-memory baseline;
- matched context/token budget;
- no full-query near duplicates;
- robust paired advantage across at least several databases/motif families.

**Kill C2 if**

- retrieval/memory closes the gap at matched context;
- gain is <2 absolute points and statistically unstable;
- gain occurs only on one motif or one database;
- manual semantic library helps strongly but automatic library does not.

---

## 3. Secondary claim

### C3 — Learned operators reduce inference complexity

Possible evidence:

- fewer generated tokens;
- fewer tool calls;
- shorter plans;
- higher first-attempt success;
- lower repair-loop count.

This claim is valid even if final accuracy is similar, but only if semantic safety is preserved.

**Do not claim** cost improvement if the library-learning/offline cost dominates the target use case without amortization analysis.

---

## 4. Representation claim

### C4 — Grain/time/schema semantics are useful canonicalization signals

This is an ablation finding, not necessarily a headline contribution.

A convincing result would identify which semantic annotations carry most of the gain and which are unnecessary.

---

## 5. Outcome matrix

| Gate A | Gate B | Interpretation | Paper direction |
|---|---|---|---|
| PASS | PASS | core thesis supported | full SemLibSQL method paper |
| PASS | FAIL | abstraction exists but does not help generation | benchmark/analysis paper on SQL semantic abstraction, or redesign interface |
| FAIL | n/a | generic library learning already sufficient | kill SemLibSQL mechanism |
| PASS only with human semantic labels | mixed | semantic layer helps, automatic induction unsolved | negative/diagnostic result, not current method paper |
| PASS with cost only | accuracy tied | language helps efficiency | efficiency paper only if savings are large and robust |
| PASS but unsafe false merges | n/a | semantic clustering too aggressive | redesign uncertainty/context scope before continuation |

---

## 6. Minimum meaningful effect sizes

These are investment thresholds, not universal statistical laws.

### Gate A

Need at least one of:

- ≥10 point absolute gain in hard-positive motif recall at similar precision;
- substantial increase in cross-realization abstraction support while holding false-merge rate low;
- several clearly useful abstractions unavailable to generic baselines, validated by downstream composition.

### Gate B

Need at least one of:

- ≥2–3 point absolute semantic execution-accuracy improvement over strongest memory baseline with consistent cross-database direction;
- similar accuracy with ≥20% reduction in tokens/tool calls/latency and no safety regression;
- much larger gain on the predeclared hard composition subset.

---

## 7. Explicit non-claims

The project does not claim:

- complete SQL equivalence checking;
- first program library learning;
- first use of e-graphs for abstraction learning;
- first SQL workload pattern mining;
- first semantic memory for Text2SQL;
- first compositional Text2SQL benchmark;
- first automatic semantic-layer generation;
- universal portability of warehouse-specific operators.

---

## 8. Failure is informative

### If generic ReGAL/Stitch wins

Finding: SQL semantics may not require a specialized abstraction layer for this workload. Valuable engineering result, but the SemLibSQL paper thesis fails.

### If oracle library wins but automatic induction fails

Finding: the bottleneck is abstraction induction, not composition. Future work should focus on semantic canonicalization/abstraction discovery rather than agent orchestration.

### If automatic library helps only repeated queries

Finding: library is functioning as compressed retrieval, not compositional language learning. Kill the core claim.

### If semantic operators cause negative transfer

Finding: learned abstraction requires explicit context/scoping/versioning before it can be trusted in long-lived database agents.

---

## 9. Review status

This document is an internal adversarial pre-registration. Formal cross-family research-review acceptance remains unavailable in the current connector environment, so no acceptance receipt should be inferred.