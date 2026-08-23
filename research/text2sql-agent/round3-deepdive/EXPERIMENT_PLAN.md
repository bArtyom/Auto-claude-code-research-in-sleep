# Experiment Plan — SemLibSQL

> **Date:** 2026-08-23  
> **Workflow role:** `experiment-plan`  
> **Goal:** falsify or validate the narrow thesis before building a large agent system.

## 0. Thesis under test

> **A semantics-aware library learner can induce reusable warehouse-specific operators from verified SQL histories that improve held-out compositional Text2SQL beyond strong episodic query memory.**

The experiment suite is deliberately ordered so the cheapest tests can kill the idea before model training or large engineering investment.

---

# 1. Hypotheses

## H1 — Semantic normalization exposes reusable structure missed by syntax

Across multiple SQL implementations of the same business motif, semantic canonicalization produces tighter/high-purity equivalence groups than token/AST similarity or syntactic library learning.

### Falsification

If syntactic methods recover the same abstractions with similar purity and downstream utility, the proposed SQL-specific mechanism is unnecessary.

## H2 — Learned semantic operators improve held-out composition

When complete query compositions are withheld, SemLibSQL outperforms verified-query retrieval/structured trace memory by composing induced primitives.

### Falsification

If retrieval/memory matches SemLibSQL under matched context and tool budget, the central thesis fails.

## H3 — Automatic induction approaches a manually authored primitive library

A small oracle semantic library should be an upper bound. Automatic induction should recover a meaningful fraction of its benefit.

### Falsification

If manual primitives help strongly but automatic induction provides little benefit, the paper becomes "automatic semantic-layer induction remains unsolved," not a successful method paper.

## H4 — Library reuse does not create unacceptable negative transfer

Optional/scoped primitive use should improve aggregate performance without increasing silent error on tasks outside the learned primitive's support.

### Falsification

If learned primitives frequently overgeneralize, the library is unsafe and requires a different uncertainty/versioning mechanism before scaling.

---

# 2. Phase A — 48-hour abstraction feasibility pilot

This is the first gate. Do not build the full Text2SQL controller yet.

## 2.1 Build a controlled corpus

Create 8–12 semantic motifs. Suggested initial set:

1. `latest_snapshot(entity, as_of)`;
2. `dedup_events(key, event_id)`;
3. `active_entity(as_of, status_policy)`;
4. `fiscal_period(date, calendar)`;
5. `bridge_join(entity_a, entity_b, bridge)`;
6. `recognized_revenue(period, currency)`;
7. `eligible_population(context)`;
8. `scd2_state(entity, as_of)`;
9. `retention_cohort(anchor, horizon)`;
10. `rolling_window(metric, width)`;
11. `topk_with_ties(entity, metric, k)`;
12. `conversion_funnel(stage_order)`.

For each motif, produce 10–25 verified SQL instances across different tables/schema names and, where possible, different SQL formulations.

Target: **200–500 verified programs**.

Use at least 3 database schemas/projects. Include two dialect realizations for a subset if cheap.

## 2.2 Ensure syntactic diversity

For the same motif, deliberately include alternative implementations:

- window function vs aggregate-and-join;
- CTE vs nested query;
- engine-specific `QUALIFY` vs subquery;
- direct relation vs precomputed model;
- `DISTINCT ON` vs row-number approach where dialect allows;
- equivalent filter pushdown/reordering.

Without this, semantic canonicalization has nothing meaningful to prove.

## 2.3 Pilot methods

### M0 — lexical/token clustering

Embed/text-similarity or token overlap over SQL.

### M1 — AST structural clustering

Canonicalize aliases/formatting only and cluster syntax trees.

### M2 — syntactic library learner

Use a Stitch-like corpus compression objective over ASTs / relational syntax.

### M3 — semantic-plan normalization

Normalize entity/grain/join/filter/aggregation/time/window structure using safe rules and schema annotations.

### M4 — semantic-plan + behavior signatures

Add small discriminating executions / invariant signatures to merge implementations that are likely behaviorally equivalent.

## 2.4 Pilot metrics

For each method:

- motif cluster purity;
- motif recall: fraction of true instances brought into the same abstraction family;
- abstraction compression ratio;
- parameter consistency;
- manual semantic-coherence score on top abstractions;
- number of useful abstractions discovered at fixed library size.

### Gate A

**PASS** if M3/M4 materially improves purity/recall or discovers semantic abstractions that M1/M2 misses.

**FAIL** if M2 is effectively equal to M3/M4. If failed, do not proceed with a claim of SQL-specific semantic library induction.

---

# 3. Phase B — Held-out motif-composition benchmark

This is the anchor paper experiment.

## 3.1 Split design

Let motifs be A, B, C, ... . Construct training/library history so primitives are individually observable but some full combinations never occur.

Example:

```text
history:
A
B
C
A+B
B+C
D+E

held-out:
A+C
A+B+C
A+D
B+D+E
```

The same physical schema should not trivially reveal the answer via memorized SQL.

## 3.2 Leakage controls

For each held-out task:

- no exact SQL duplicate in history;
- no near-duplicate complete query after normalized AST comparison;
- no identical NL template with only literal substitutions;
- report nearest historical SQL similarity;
- optionally withhold the exact target schema subgraph combination from any single historical query.

## 3.3 Systems compared

### B0 — Stateless strong Text2SQL

Strong base model + same schema/context.

### B1 — Nearest verified-query retrieval

Retrieve top-k prior `(NL, SQL)` examples.

### B2 — Retrieval with maximal fair context budget

Give B1 as much historical context as SemLibSQL consumes in operator descriptions/definitions.

### B3 — Structured trace/semantic memory

AgentSM-like memory: retrieve structured prior solution plans rather than raw SQL.

### B4 — Crystallized verified-query memory

Store corrected/verified successful SQL and reuse database-specific examples.

### B5 — Manual/oracle semantic library

Human-authored correct primitives for the same motifs. This is an upper-bound diagnostic.

### B6 — Syntactic learned library

Library induced without semantic canonicalization.

### B7 — **SemLibSQL**

Semantics-aware induced library.

## 3.4 Primary metrics

1. **semantic execution accuracy** on held-out compositions;
2. **first-attempt success** before repair loops;
3. **compositional generalization gap** relative to tasks whose full composition appeared in history;
4. tokens;
5. tool/DB calls;
6. end-to-end latency if measurable.

## 3.5 Statistical plan

- report paired per-task outcomes across systems;
- bootstrap confidence intervals for accuracy deltas;
- use a paired test appropriate for binary correctness (e.g. McNemar on matched tasks) in final analysis;
- report per-motif-composition breakdown, not only aggregate mean;
- predeclare the main comparison: **B7 vs strongest of B2/B3/B4**.

### Gate B

Proceed to a full paper implementation only if SemLibSQL shows a robust advantage on held-out compositions or a compelling accuracy-cost Pareto improvement.

Suggested practical threshold for continued investment:

- **≥2–3 absolute points** over the strongest memory baseline with consistent direction across databases, **or**
- similar accuracy with a substantial inference-cost reduction while preserving semantic safety.

A tiny noisy mean gain is insufficient.

---

# 4. Phase C — Semantic abstraction audit

The paper must prove the learned library is not just compressed syntax.

## 4.1 Purity

Sample each learned primitive's supporting instances. Human annotators label whether they express one coherent semantic motif.

Metric: weighted and macro primitive purity.

## 4.2 Parameter invariance

Substitute:

- entity/table;
- time window;
- dimension;
- business unit;
- dialect implementation;

and verify that the abstraction behaves according to its declared parameters.

## 4.3 Intervention tests

For a primitive `latest_snapshot`, construct data where:

- newest row is duplicated;
- timestamps tie;
- future rows exist;
- null timestamps exist;
- multiple business keys share values.

Check behavior against the operator contract.

## 4.4 Description/name independence

Ablate LLM-generated names/docstrings. If gains disappear because the base model needs semantically helpful names, report this honestly; naming may be part of the interface, but it should not be confused with abstraction discovery.

---

# 5. Phase D — Mechanism ablations

## A1 — No semantic canonicalization

Mine from normalized SQL syntax only.

## A2 — Safe rewrites only

No execution/behavior signatures.

## A3 — No type/grain annotations

Tests whether semantic typing contributes.

## A4 — No abstraction induction

Retrieve canonical historical plans directly.

This is essential: it isolates library invention from representation improvement.

## A5 — No primitive validation

Promote all frequent abstractions. Measure purity and negative transfer.

## A6 — Force library use

Every task must use at least one primitive.

Compare with optional routing. This quantifies negative transfer and validates fallback behavior.

## A7 — Library-size sweep

Small/medium/large libraries to test whether more abstractions monotonically help or create interference.

---

# 6. Phase E — Real / enterprise-like workload

Controlled motifs are necessary but insufficient.

Candidate sources:

- Spider 2.0 / repository-style enterprise tasks where reuse across one environment can be constructed;
- open dbt projects with recurring models/macros and query tasks;
- public analytical SQL workloads;
- a manually curated subset of BIRD-like schemas with repeated business motifs;
- generated task streams grounded in real open schemas, with human-audited semantic contracts.

## Required procedure

1. identify recurring semantic motifs without using test outcomes;
2. create chronological or compositional history/test split;
3. deduplicate near-identical full queries;
4. audit a subset for motif correctness;
5. compare memory vs learned library under matched context.

### Gate E

Broad enterprise claims require at least one real/enterprise-like positive result. If only the controlled benchmark works, position the paper narrowly as a program-learning study.

---

# 7. Phase F — Negative transfer and safety

## 7.1 Context-dependent motif test

Create concepts with the same surface name but different semantics:

```text
active_customer_US
active_customer_EU
revenue_finance
revenue_sales_ops
```

Test whether the learned library scopes them correctly or incorrectly merges them.

## 7.2 Distribution shift

Change:

- schema version;
- business rule;
- dialect;
- time period;
- cardinality pattern.

Measure stale primitive use.

## Metrics

- wrong-primitive routing rate;
- silent-error rate attributable to library;
- negative-transfer delta vs stateless baseline;
- fallback success;
- per-primitive blast radius.

This phase does not require a full belief-revision system. It only tests whether the first method is safe enough to claim reuse.

---

# 8. Optional diagnostic — Cross-dialect reuse

If the semantic representation naturally separates meaning from dialect:

1. induce primitives from dialect A;
2. lower/use them in dialect B with matched schema semantics;
3. compare against raw SQL retrieval from A.

This could demonstrate that the library captures semantics rather than surface code, but it is optional and should be cut if it distracts from the anchor experiment.

---

# 9. Resource plan

## Stage 1 — very cheap

- offline parsing/canonicalization;
- 200–500 programs;
- fixed LLM only for optional naming/semantic annotations;
- CPU + small DB execution;
- goal: Gate A.

## Stage 2 — cheap/moderate

- 300–1000 composition tasks;
- 3–5 baseline systems;
- fixed frontier/specialist model under controlled budget;
- goal: Gate B.

## Stage 3 — only after positive signal

- real workload;
- more databases/dialects/models;
- human abstraction audit;
- full ablations/statistics.

No model fine-tuning or RL before Gates A and B.

---

# 10. Decision table

| Result | Decision |
|---|---|
| Semantic normalization ≈ syntactic mining | Kill SQL-specific mechanism; reconsider generic library-learning paper |
| Normalization better, but library ≈ memory | Interesting representation result, core composition thesis fails |
| Library > memory on controlled split only | Continue cautiously; real-workload gate mandatory |
| Library > memory on controlled + real workload | Strong proceed signal |
| Manual library >> learned library | Semantic operators useful; automatic induction needs more work |
| Learned library improves mean but creates high silent negative transfer | Do not claim reliable agent; solve scoping/validation first |
| Similar accuracy but 30–50% lower inference cost | Potential efficiency contribution if robust and library induction cost amortizes |

---

# 11. Minimal run order

1. **R1:** hand-label 8–12 motifs and collect 200–500 verified SQL instances.
2. **R2:** lexical/AST/syntactic clustering baselines.
3. **R3:** semantic normalization + cluster/abstraction purity test.
4. **Gate A.**
5. **R4:** construct leakage-controlled held-out compositions.
6. **R5:** stateless + retrieval + structured-memory baselines.
7. **R6:** SemLibSQL induced library.
8. **Gate B.**
9. **R7:** manual/oracle library and mechanism ablations.
10. **R8:** real/enterprise-like workload.
11. **R9:** negative-transfer stress tests.
12. Only then consider larger model/training/system extensions.

---

# 12. Paper-level must-prove package

A credible paper needs four figures/tables:

### Figure 1 — Method

Verified query history -> semantic normalization -> abstraction induction -> validated library -> new composition.

### Figure 2 — Composition benchmark

Accuracy vs degree of unseen motif composition for memory vs syntactic library vs SemLibSQL.

### Figure 3 — Accuracy/cost curve over accumulated experience

Does the library reduce future reasoning as experience grows?

### Table 1 — Main results

Controlled composition + real workload, with strong memory baselines.

### Table 2 — Mechanism/semantic-quality ablations

Canonicalization, typing, validation, optional routing, abstraction purity, negative transfer.

---

# 13. Final experiment gate

```yaml
first_run: abstraction_feasibility
large_scale_implementation: blocked_until_gate_A_and_B
training_new_model: not_needed_for_pilot
main_baseline: strongest_verified_query_memory
primary_metric: held_out_composition_semantic_accuracy
critical_safety_metric: negative_transfer_silent_error
```

The project should treat a clean negative pilot as a success of the research process: it prevents months of engineering on a library mechanism that modern memory systems can already match.