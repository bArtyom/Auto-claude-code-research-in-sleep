# Research Contract: SemLibSQL-Γ

**Paper-facing working title:** **SemLibSQL-Γ: Learning Guarded Semantic Operators from Verified Text-to-SQL Workloads**  
**Date:** 2026-08-23  
**State:** idea selected and frozen; empirical validation not started.

## Selected Idea

SemLibSQL-Γ studies whether repeated warehouse SQL contains a **Guarded Abstraction Gap (GAG)**: reusable semantic operations that generic syntax/global-theory library learners fail to recover safely because equivalence depends on local warehouse contracts. The system learns/reuses an `(operator, guard)` pair rather than an unconditional macro.

Example:

```text
operator:
  latest_snapshot(relation, key, effective_time, as_of)

guard:
  key has the target business grain
  effective_time is the intended validity order
  as_of excludes future rows
  tie semantics are resolved
```

The active research question is not “can we learn functions from SQL?”; generic library learning already does that. It is whether **warehouse-conditioned guards materially change abstraction discovery and safe compositional reuse**.

## Selection Rationale

This direction survived repeated filtering of active-schema agents, generic memory, semantic layers, result-feedback repair, ambiguity/clarification, test-time search, and broad long-horizon-agent ideas. Its final novelty is modest because the closest PL components are strong, but it is cheap to falsify and has a clear scientific fork:

- no Guarded Abstraction Gap → kill;
- gap exists but generic PL composition handles it → benchmark/analysis result only;
- gap exists and joint guarded induction helps → method paper candidate.

## Core Claims

1. **C1 — Guarded Abstraction Gap:** Warehouse-conditioned equivalence recovers semantically coherent hard positives at matched false-merge precision beyond strong fixed-theory/global library-learning baselines.
2. **C2 — Guard safety:** Applicability guards reject near-miss reuse when a relevant warehouse condition is violated.
3. **C3 — Composition beyond memory:** Guarded operators improve held-out combinations of known motifs over strong verified-query/structured-memory baselines.
4. **C4 — Joint induction (optional):** Joint abstraction+guard selection improves over a pipeline that separately learns e-graph abstractions and then infers preconditions.

## Closest Prior Work That Must Be Treated as Baselines

### Program-library learning

- DreamCoder
- Stitch
- babble / Library Learning Modulo Theory (LLMT)
- ReGAL
- **E-Stitch (EGRAPHS 2026)** — top-down Stitch-style library learning directly over e-graphs

### Conditional reasoning / guard inference

- classical conditional rewriting
- Colored E-Graphs
- **Predicate E-Graphs with Symbolic Conditional Rewriting (EGRAPHS 2026)**
- Combining E-Graphs with Abstract Interpretation
- **Alive-Infer (PLDI 2017)** — data-driven precondition inference for optimization rules
- general precondition-inference literature

### SQL/database side

- SQL equivalence under integrity constraints
- U-semiring-style SQL equivalence
- VeriEQL
- workload-driven constraint/rewrite discovery

### Text2SQL reuse

- AgentSM
- GATE
- semantic-layer-mediated Text2SQL
- Snowflake Semantic View Autopilot

## Method Summary

### Warehouse contract Γ

Each fact carries provenance:

- `DECLARED`: explicit schema/dbt/semantic contract;
- `VERIFIED`: supported by a strong deterministic/formal/test check;
- `EMPIRICAL`: observed in data but not guaranteed;
- `HYPOTHESIS`: unsafe for autonomous library reuse.

The first experiment should permit only `DECLARED` + `VERIFIED` facts to certify reuse, with empirical facts as a later ablation.

### Candidate equivalence

For candidate SQL plans `P` and `Q`, seek evidence for `P ≡_Γ Q` using:

1. safe relational normalization;
2. formal/bounded equivalence under supported constraints;
3. contract-preserving differential/counterexample database instances;
4. grain/lineage/invariant checks.

Result is always one of:

```text
EQUIVALENT
NOT_EQUIVALENT
UNKNOWN
```

`UNKNOWN` must be preserved.

### Guarded abstraction induction

For each candidate recurring family, learn/derive:

```text
semantic skeleton
parameters
supporting programs
compact sufficient guard
known counterexamples
constraint provenance
```

The system must not present the guard as novel precondition-inference technology. The novelty bet is its **workload-conditioned use as the organizing object for Text2SQL abstraction learning and composition**.

### Inference

Use a learned operator only if the current context satisfies its guard; otherwise fall back to the base Text2SQL generator.

## Minimum Convincing Evidence

### Gate A — the only immediate experiment

Show a real Guarded Abstraction Gap against:

- token/AST baselines;
- Stitch;
- babble/LLMT;
- **E-Stitch**;
- ReGAL;
- a **composed conditional-PL baseline**: e-graph library learning + predicate/conditional reasoning + Alive-Infer-style guard inference.

Need hard positives where global/syntactic theory cannot safely merge implementations and hard negatives where a missing guard changes semantics.

### Gate B

Only if A passes: held-out motif composition versus matched-context query retrieval, AgentSM-like structured memory, GATE-like grounding memory, strongest generic library baseline, and an oracle/manual guarded library.

## Kill Conditions

Kill the current method thesis if:

1. a strong E-Stitch/babble/ReGAL baseline recovers the same abstraction frontier;
2. the composed conditional-PL baseline matches joint SemLibSQL-Γ;
3. improvements reduce to superficial SQL normalization;
4. false semantic merges rise materially;
5. useful guards require per-motif manual authoring;
6. verified/structured memory matches held-out composition at equal context;
7. the Guarded Abstraction Gap is negligible on realistic workloads.

**Do not add RL, MCTS, multi-agent debate, extra critics, or memory routing to rescue a failed Gate A.**

## Experiment Design

- 8–12 motifs, 200–500 verified SQL programs, ≥3 schemas/projects.
- Hard positives: semantically same only under specified warehouse laws; radically different SQL realization.
- Hard negatives: visually/structurally near but one law is violated (grain, uniqueness, coverage, nullability, time/tie policy, metric definition).
- Primary Gate-A metrics: hard-positive recall at fixed high precision, false-merge rate, cross-realization abstraction support, guard compactness/accuracy, UNKNOWN rate.
- Gate-B metrics: semantic execution accuracy, first-attempt success, compositional gap, tokens/tools, negative transfer.
- No large-model training is required before Gate A determines whether the idea deserves further work.

## Backup Direction

If Gate A kills SemLibSQL-Γ, switch to **TemporalSQL-Drift**: a Text2SQL benchmark for changing business semantics that distinguishes valid time, knowledge time, and recomputation under current definitions. General bitemporal memory is prior art; the contribution would be evaluation of business-semantic drift/historical reproducibility in database agents.

## Status

- [x] broad literature survey
- [x] repeated novelty filtering
- [x] final last-mile PL/database novelty stress test
- [x] active idea selected
- [x] internal adversarial review
- [x] final proposal
- [x] pre-registered experiment design
- [ ] independent external reviewer acceptance
- [ ] Gate-A corpus execution
- [ ] Gate-A result
- [ ] Gate-B result
- [ ] implementation / paper

## Next-Step Pointer

`research/text2sql-agent/refine-logs/EXPERIMENT_PLAN.md`

**Stop before execution:** the next unresolved question is empirical.
