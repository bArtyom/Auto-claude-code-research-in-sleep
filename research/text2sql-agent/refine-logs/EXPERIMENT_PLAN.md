# Experiment Plan — SemLibSQL-Γ

**Date:** 2026-08-23  
**Status:** PRE-REGISTERED DESIGN; NO EXPERIMENTS RUN  
**Purpose:** make the next step executable while preserving the user's stop condition at the boundary where empirical work begins.

---

## 0. Thesis under test

> A warehouse-conditioned library learner can discover reusable, scoped semantic SQL operators that strong generic library/refactoring methods miss, and those operators improve held-out Text2SQL composition beyond structured memory.

The plan is intentionally staged. **Do not build a full agent before Gate A passes.**

---

# 1. Hypotheses

## H1 — Contextual equivalence changes abstraction discovery

Programs that are equivalent only under warehouse-specific constraints can be safely grouped into reusable motif families more effectively by SemLibSQL-Γ than by syntax/global-theory baselines.

### Falsification

If Stitch/babble/ReGAL-style baselines with generous SQL rewrite rules recover the same hard-positive families at similar false-merge rate, the SQL-specific mechanism is unnecessary.

## H2 — Scope contracts prevent negative transfer

Attaching applicability conditions to abstractions reduces unsafe reuse on near-miss contexts where one relevant warehouse law is violated.

### Falsification

If scoped and unscoped libraries have similar false-reuse behavior, scope contracts are not a meaningful mechanism.

## H3 — Learned operators enable composition beyond memory

On target queries whose motifs were individually observed but whose complete combination was withheld, SemLibSQL-Γ outperforms the strongest verified-query/structured-memory baseline under matched context budget.

### Falsification

If memory/retrieval matches SemLibSQL-Γ, the library is just compressed episodic memory.

---

# 2. Gate A — Contract-Conditioned Abstraction Feasibility

**This is the first real experiment and the exact point where this idea-discovery run stops.**

## 2.1 Corpus target

Create **200–500 verified SQL programs** covering 8–12 motifs across at least 3 schemas/projects.

Recommended motif pool:

1. `latest_snapshot(entity, as_of)`
2. `scd2_state(entity, as_of)`
3. `deduplicate(key, event_id)`
4. `bridge_join(entity_a, entity_b, bridge)`
5. `fiscal_period(date, calendar)`
6. `recognized_revenue(period, currency)`
7. `eligible_population(context)`
8. `active_entity(as_of, status_policy)`
9. `retention_cohort(anchor, horizon)`
10. `rolling_window(metric, width)`
11. `topk_with_ties(entity, metric, k)`
12. `conversion_funnel(stage_order)`

The earlier seed specifications in `round4-semlibsql-pilot/SEED_CORPUS_MANIFEST.md` can initialize this corpus, but they must be instantiated into executable SQL/database states before they count as evidence.

## 2.2 Hard-positive design

For each motif, include genuinely distinct implementation families, e.g.:

- window + row-number vs aggregate-and-join;
- `QUALIFY` vs nested subquery;
- `DISTINCT ON` vs row-number;
- direct base-table formulation vs equivalent dbt/logical view;
- join path collapsed by a validated relationship;
- semantically equivalent filter/order rewrites;
- different dialect realizations for a subset.

A hard positive is useful only when generic AST similarity is low enough that trivial normalization does not solve it.

## 2.3 Hard-negative design

Construct near-miss pairs that look similar but differ under one critical semantic condition:

- `latest before as_of` vs absolute latest;
- uniqueness holds vs duplicate-key case;
- `COUNT(*)` vs `COUNT(DISTINCT entity_id)` under violated uniqueness;
- inner vs left join when existence/totality fails;
- one row per order vs one row per customer;
- inclusive vs exclusive temporal boundary;
- different tie-handling policy;
- nullable vs non-nullable join/filter semantics;
- gross vs recognized/net metric;
- current snapshot vs valid-time snapshot.

These examples are mandatory because pure recall can reward unsafe over-merging.

## 2.4 Warehouse contracts

For every test family, record facts as:

```text
DECLARED
VERIFIED
EMPIRICAL
HYPOTHESIS
```

Gate A should initially allow only `DECLARED` and `VERIFIED` facts to certify automatic reuse. Run an ablation later with empirical constraints.

## 2.5 Methods

### A0 — Token / lexical similarity

Normalized SQL text/embedding clustering.

### A1 — Normalized AST

Alias/format/literal normalization plus tree similarity.

### A2 — Stitch-style syntactic library learning

Library induction over normalized SQL/relational syntax.

### A3 — babble/LLMT-style fixed-theory library learning

Give the baseline a generous, manually curated set of universally safe SQL/relational equalities. This is the **critical baseline**.

### A4 — ReGAL-style execution-validated refactoring

Use execution to validate reusable refactorings without warehouse-specific scope contracts.

### A5 — SemLibSQL with global semantics only

Same architecture as the proposed method, but no warehouse-specific conditional laws.

### A6 — **SemLibSQL-Γ**

Use contract-conditioned equivalence plus scoped operators.

## 2.6 Metrics

Primary abstraction metrics:

- hard-positive motif recall;
- hard-positive precision / cluster purity;
- **false semantic merge rate** on hard negatives;
- precision–recall curve or recall at fixed high precision;
- number of cross-realization motif families recovered;
- support size and parameter consistency per abstraction.

Scope metrics:

- precondition violation detection rate;
- false reuse with scope vs without scope;
- fraction of operators with compact/nontrivial scope contracts;
- manual effort required to define/validate scope.

Secondary:

- compression ratio;
- induction time;
- equivalence-oracle UNKNOWN rate;
- fraction of decisions certified formally vs by differential tests.

## 2.7 Gate-A decision rule

### PASS

Continue only if at least one of the following is clear and reproducible:

- SemLibSQL-Γ improves hard-positive recall by roughly **≥10 absolute points** at comparable high precision / false-merge rate versus the strongest generic baseline; or
- it discovers several high-support, semantically coherent cross-realization abstractions that fixed-theory LLMT/ReGAL cannot recover and these are later usable for composition; or
- scope contracts produce a material safety frontier improvement that generic/unscoped libraries cannot match.

### FAIL

Kill the core mechanism if:

- A3/A4 ≈ A6;
- gains reduce to formatting/CTE/alias normalization;
- false merges rise materially;
- most useful contracts are manually supplied at motif level;
- equivalence remains UNKNOWN for too much of the realistic workload to support useful induction.

**Do not add agent complexity after a Gate-A failure.**

---

# 3. Gate B — Held-Out Motif Composition

Run only after Gate A passes.

## 3.1 Split design

Construct a history where motifs are individually present and some combinations are observed, while target combinations are absent.

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

with motifs such as:

```text
A = latest_snapshot
B = active_entity
C = recognized_revenue
D = fiscal_period
E = eligible_population
```

## 3.2 Leakage controls

For each target:

- no exact SQL duplicate;
- no near-duplicate complete normalized plan;
- no NL template with literal-only substitutions;
- report nearest historical SQL/plan similarity;
- where possible, no single historical query contains the full target schema subgraph + operator combination;
- include cross-dialect/cross-schema variants for a subset.

## 3.3 Systems

### B0 — Strong stateless Text2SQL

Same base model, schema/context, no history.

### B1 — Verified-query retrieval

Top-k question/SQL examples.

### B2 — Matched-context retrieval

Allow as much historical text as the proposed operator definitions consume.

### B3 — AgentSM-style structured memory

Retrieve structured solution programs/traces.

### B4 — Execution-grounded semantic memory

GATE-like reusable grounding facts where applicable.

### B5 — ReGAL/babble learned generic library

Strongest Gate-A generic abstraction baseline.

### B6 — Manual/oracle semantic library

Human-authored correct motif operators; diagnostic upper bound.

### B7 — **SemLibSQL-Γ**

Automatically induced scoped library.

## 3.4 Primary metrics

- semantic execution correctness;
- first-attempt success before repair loops;
- compositional generalization gap;
- token use;
- DB/tool calls;
- end-to-end latency if stable;
- negative-transfer rate on out-of-scope tasks.

## 3.5 Statistical plan

- predeclare B7 vs strongest of B2/B3/B4/B5 as main comparison;
- paired per-task outcomes;
- bootstrap CIs for accuracy deltas;
- McNemar test for paired binary correctness;
- report per-database and per-composition family;
- include nearest-history similarity as a covariate/diagnostic.

## 3.6 Gate-B investment threshold

Continue toward a full paper only if:

- **≥2–3 absolute points** over the strongest memory/generic-library baseline with consistent direction across databases, or
- similar accuracy with **≥20%** meaningful inference-cost reduction and no safety regression, or
- a substantially larger, predeclared gain on the hard compositional subset.

Tiny aggregate gains are insufficient.

---

# 4. Gate C — Scope and Safety Audit

Run if Gate B is promising.

For each learned operator, deliberately violate one contract at a time:

- uniqueness;
- key coverage;
- nullability;
- time/tie policy;
- relationship totality;
- grain;
- metric definition;
- data-version assumptions.

Measure whether the system:

1. refuses the operator;
2. falls back safely;
3. incorrectly reuses it;
4. propagates the mistake to a final SQL answer.

This is necessary to distinguish useful contextual abstraction from unsafe pattern reuse.

---

# 5. Required Ablations

1. **No warehouse contract** — global safe rewrites only.
2. **No scope contract** — learn abstraction but reuse everywhere.
3. **Declared constraints only** vs declared + verified inferred constraints.
4. **No formal equivalence component** — differential execution only.
5. **No differential counterexamples** — static/formal evidence only.
6. **No semantic types/grain** — relational plan only.
7. **No operator names/docstrings** — test whether gains come merely from helpful language descriptions.
8. **Syntactic library with same size/token budget** — fairness control.

---

# 6. Reporting Discipline

If Gate A fails, the correct conclusion is:

> generic library learning is sufficient for the tested SQL reuse regime, or the proposed contextual-equivalence machinery does not justify its complexity.

If oracle library succeeds but automatic induction fails:

> semantic abstraction is useful, but automatic abstraction/scope induction remains unsolved.

If Gate A passes but Gate B fails:

> contextual abstraction exists but does not improve Text2SQL generation beyond memory; consider an analysis/benchmark paper, not the current method claim.

If scope safety fails:

> do not deploy or claim reliable long-lived reuse; redesign scoping/versioning first.

---

# 7. Experiment Start Checklist

Before running Gate A:

- [ ] instantiate executable database schemas/states;
- [ ] instantiate 200–500 SQL cases with motif labels;
- [ ] independently verify gold motif and hard-negative labels;
- [ ] record contract facts and provenance classes;
- [ ] implement/obtain A0–A4 baselines;
- [ ] freeze scoring code and primary metrics;
- [ ] freeze Gate-A thresholds;
- [ ] log random seeds / DB engine versions;
- [ ] ensure no LLM judge is the sole correctness oracle.

**Current status:** none of these empirical steps has been executed in this idea-stage session.
