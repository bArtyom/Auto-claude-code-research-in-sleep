# Experiment Plan — SemLibSQL-Γ

**Date:** 2026-08-23  
**Status:** **PRE-REGISTERED DESIGN; NO EXPERIMENTS RUN**  
**Purpose:** falsify the Guarded Abstraction Gap before building a Text2SQL agent.

---

## 0. Thesis under test

> Realistic verified SQL workloads contain reusable operations whose equivalence is **warehouse-conditional**. Jointly learning guarded abstractions can recover/use that structure more safely or effectively than strong global-theory library learning, separate precondition inference, and episodic Text2SQL memory.

This plan is ordered so the cheapest evidence can terminate the idea early.

---

# 1. Hypotheses

## H1 — Guarded Abstraction Gap exists

At matched high precision, conditioning on local warehouse contracts recovers hard-positive semantic motif families missed by syntax/global fixed-theory library learning.

**Falsify if:** strong E-Stitch/babble/ReGAL baselines with generous SQL rewrite theory recover essentially the same families.

## H2 — Guards matter for safety

Explicit applicability guards substantially reduce false reuse on near-miss contexts where one semantic condition is violated.

**Falsify if:** scoped vs unscoped reuse has negligible difference, or guards are effectively hand labels.

## H3 — Joint induction adds value beyond a composed PL pipeline

Joint abstraction+guard selection improves over:

```text
E-Stitch/babble-style abstraction
→ predicate/conditional reasoning
→ Alive-Infer-style precondition inference
```

**Falsify if:** the composed baseline matches the joint method. If H1 still holds, only a benchmark/analysis paper remains plausible.

## H4 — Guarded library beats memory on unseen composition

On withheld combinations of known motifs, SemLibSQL-Γ outperforms strongest verified-query/structured-memory baselines under matched context.

**Falsify if:** memory closes the gap.

---

# 2. Gate A — Guarded Abstraction Gap

**This is the first real experiment. The current idea-discovery session stops before executing it.**

## 2.1 Corpus

Target **200–500 verified SQL programs**, 8–12 motifs, at least 3 schemas/projects.

Suggested motifs:

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

The existing `round4-semlibsql-pilot/SEED_CORPUS_MANIFEST.md` is specification-only. Cases count only after they become executable and independently checked.

## 2.2 Hard positives

Require genuinely different implementations, for example:

- window/row-number vs aggregate-and-join;
- `QUALIFY` vs nested subquery;
- `DISTINCT ON` vs row-number;
- base-table logic vs validated logical/dbt view;
- alternate join paths equivalent only under declared relationships;
- equivalent aggregation forms under a key/grain contract;
- multiple dialect realizations.

Hard positives must be difficult for normalized AST similarity.

## 2.3 Hard negatives

Near-identical programs must become non-equivalent after one contract fact is changed:

- unique key vs duplicate key;
- total relationship vs missing dimension row;
- non-null vs nullable key;
- latest-before-`as_of` vs absolute latest;
- unique timestamp vs tied timestamp;
- customer grain vs order grain;
- inner vs left join under violated coverage;
- inclusive vs exclusive date boundary;
- gross vs recognized/net metric;
- current-state vs valid-time state.

These cases measure safety, not just compression.

## 2.4 Contract fact classes

For every candidate fact record:

```text
DECLARED   # schema/dbt/governed contract
VERIFIED   # deterministic/formal/test evidence
EMPIRICAL  # observed in current data only
HYPOTHESIS # unsupported candidate
```

Primary Gate A allows `DECLARED` + `VERIFIED` facts to certify reuse. Later ablate empirical facts.

---

# 3. Gate-A systems

## A0 — lexical/token baseline

Normalized SQL tokens / embeddings.

## A1 — normalized AST/relational plan

Aliases/formatting/literals plus conservative structural normalization.

## A2 — Stitch

Strong syntax/corpus library learning.

## A3 — babble / LLMT

Library learning modulo a generous manually supplied globally valid SQL/relational equational theory.

## A4 — **E-Stitch (2026)**

Top-down library learning directly over e-graphs. This is now a mandatory baseline, not optional related work.

## A5 — ReGAL-style execution refactoring

Execution-validated reusable function learning without warehouse-specific guarded-library objective.

## A6 — Conditional equality baseline

Use colored/predicate-egraph-style assumptions or equivalent conditional rewriting with a fixed set of guards.

## A7 — Separate guard inference baseline

First learn candidate abstractions with A3/A4/A5, then infer applicability conditions using an **Alive-Infer-style** positive/negative precondition learner.

This is the critical “obvious composition of prior art” baseline.

## A8 — SemLibSQL-global

Same implementation/search machinery as proposed method but only globally valid theory; no local warehouse-conditioned guards.

## A9 — **SemLibSQL-Γ joint**

Jointly score/select abstraction and compact applicability guard for workload reuse and downstream utility.

---

# 4. Gate-A metrics

## 4.1 Guarded Abstraction Gap metrics

- hard-positive motif recall;
- precision / semantic cluster purity;
- **false semantic merge rate** on hard negatives;
- recall at fixed high precision (primary view);
- number of cross-realization families recovered;
- abstraction support size;
- parameter consistency.

Define a reported **GAG delta** such as:

```text
best recall at precision ≥ P under warehouse-conditioned guards
-
best recall at precision ≥ P under strongest global/fixed-theory baseline
```

Report across multiple `P` values rather than one arbitrary threshold.

## 4.2 Guard quality

- guard precision/recall against known validity contexts;
- near-miss rejection rate;
- false rejection of valid contexts;
- guard complexity (predicates/atoms);
- proportion derived automatically;
- provenance composition (`DECLARED` vs `VERIFIED` etc.);
- generalization to a held-out schema/context for the same motif.

## 4.3 Oracle/equivalence diagnostics

- `EQUIVALENT / NOT_EQUIVALENT / UNKNOWN` rate;
- fraction resolved by safe rewrite vs formal/bounded solver vs differential execution;
- time/cost per judgment;
- counterexample yield.

Compression ratio is secondary only.

---

# 5. Gate-A decision tree

## FAIL-0 — no phenomenon

If the Guarded Abstraction Gap is negligible versus A3/A4/A5:

> **Kill SemLibSQL entirely.**

No agent architecture should be added.

## FAIL-1 — phenomenon exists, but prior-art composition solves it

If local guards reveal a real gap but A7 (separate E-Stitch/LLMT + precondition inference) matches A9:

> **Drop the joint-method novelty claim.**

Possible remaining output: Guarded Abstraction Gap benchmark/analysis paper, only if the phenomenon is large, realistic, and informative.

## PASS-A — joint method matters

Proceed to Gate B only if A9 materially improves the precision–recall/safety frontier over **both** strong global theory (A3/A4) and prior-art composition (A7).

Practical investment threshold: look for at least one of:

- roughly **≥10 absolute points** hard-positive recall at similar high precision;
- clearly superior GAG frontier across several motifs/databases;
- materially safer guard generalization at comparable coverage;
- several high-support abstractions inaccessible to baselines and demonstrably useful downstream.

A tiny mean gain is not enough.

---

# 6. Gate B — held-out composition beyond memory

Run only after PASS-A or a deliberate analysis-paper pivot.

## 6.1 Split

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

Motifs can be:

```text
A latest_snapshot
B active_entity
C recognized_revenue
D fiscal_period
E eligible_population
```

## 6.2 Leakage controls

- no exact SQL duplicate;
- no near-duplicate complete normalized plan;
- no NL template with literal-only substitution;
- report nearest history similarity;
- where possible, no historical query contains the full target schema-subgraph/operator composition;
- cross-schema/dialect variants for a subset.

## 6.3 Systems

- **B0:** strong stateless Text2SQL;
- **B1:** top-k verified-query retrieval;
- **B2:** matched-context maximal retrieval;
- **B3:** AgentSM-like structured program memory;
- **B4:** GATE-like execution-grounded semantic memory;
- **B5:** strongest generic library baseline from Gate A;
- **B6:** separate abstraction+guard pipeline (A7) if viable;
- **B7:** manual/oracle guarded semantic library;
- **B8:** SemLibSQL-Γ joint.

## 6.4 Metrics

- semantic execution correctness;
- first-attempt success;
- compositional generalization gap;
- tokens;
- DB/tool calls;
- latency;
- out-of-scope/negative-transfer rate.

## 6.5 Statistics

- predeclare B8 vs strongest of B2/B3/B4/B5/B6;
- paired task outcomes;
- bootstrap CIs;
- McNemar for binary correctness;
- per-database/per-composition breakdown;
- nearest-history similarity diagnostic.

## 6.6 Continue threshold

Full method-paper investment requires either:

- **≥2–3 absolute execution-accuracy points** over strongest relevant baseline with consistent cross-database direction, or
- similar accuracy with **≥20%** meaningful inference-cost reduction and no safety loss, or
- a substantially larger predeclared gain on the hard composition subset.

---

# 7. Gate C — guard violation / negative transfer

Deliberately violate one guard atom at a time:

- uniqueness;
- nullability;
- key coverage;
- join cardinality;
- time/tie policy;
- grain;
- metric definition;
- data-version assumptions.

Measure whether the system refuses/falls back versus silently reusing the abstraction.

This is mandatory for any reliability claim.

---

# 8. Required ablations

1. no warehouse-local guard;
2. fixed/global equational theory only;
3. E-Stitch vs babble vs ReGAL candidate generation;
4. separate precondition inference vs joint abstraction+guard objective;
5. declared constraints only vs declared+verified inferred constraints;
6. formal/bounded equivalence removed;
7. differential counterexamples removed;
8. grain/type signals removed;
9. operator natural-language names/docstrings removed;
10. equal library size/context budget.

---

# 9. Backup trigger

If Gate A kills the SemLibSQL line, the preselected backup is **TemporalSQL-Drift** rather than adding complexity to SemLibSQL.

TemporalSQL-Drift should test questions whose correct semantics differ by:

- historical valid-time definition;
- what the organization/agent knew at knowledge time;
- current-definition recomputation.

A fresh literature search is required before executing that backup, but current search did not surface a directly matching Text2SQL benchmark.

---

# 10. Experiment-start checklist

Before any empirical claim:

- [ ] instantiate executable DB schemas/states;
- [ ] instantiate 200–500 SQL programs;
- [ ] independently verify motif/hard-negative labels;
- [ ] record guard facts + provenance;
- [ ] implement/obtain A0–A7 baselines;
- [ ] freeze scoring code and thresholds;
- [ ] record engine versions/random seeds;
- [ ] avoid LLM judge as sole correctness oracle.

**Current state:** every item above is NOT STARTED.

This is intentionally the stopping point of the idea phase.
