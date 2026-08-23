# Research Contract: SemLibSQL-Γ

> **Focused working contract for the selected Text2SQL idea.**  
> **Date:** 2026-08-23  
> **Status:** selected; implementation and experiments not started.

## Selected Idea

- **Paper-facing name:** **SemLibSQL: Learning Reusable SQL Abstractions with Warehouse-Conditioned Equivalence**
- **Internal shorthand:** SemLibSQL-Γ / ContractLibSQL
- **Description:** Learn a warehouse-specific library of reusable semantic operators from verified Text2SQL histories. Unlike ordinary query memory or generic program-library learning, candidate SQL implementations are grouped using equivalence evidence conditioned on warehouse-specific integrity/business contracts. Each induced operator carries explicit applicability conditions and evidence; it is reusable only when those conditions hold.
- **Source:** `research/text2sql-agent/idea-stage/IDEA_REPORT.md`, Idea #1
- **Selection rationale:** strongest remaining combination of novelty, falsifiability, and bounded experiment cost after eliminating active-schema, generic memory, generic semantic-layer, result-feedback, ambiguity, and environment-modification ideas with recent/older prior art.

## Core Claims

1. **Contract-conditioned abstraction discovery:** Warehouse-conditioned equivalence recovers reusable semantic motif families across heterogeneous SQL realizations better than syntactic/global-theory library learners at matched false-merge precision.
2. **Compositional generalization beyond memory:** Scoped induced operators improve held-out compositions over strong verified-query and structured-semantic-memory baselines under matched context budget.
3. **Scope safety:** Explicit operator applicability contracts reduce negative transfer on near-miss queries compared with unscoped abstraction reuse.

## Method Summary

Let `Γ` be a warehouse contract assembled from high-confidence facts such as primary/foreign keys, uniqueness, nullability, dbt tests/contracts, validated lineage, and evidence-backed business invariants. SemLibSQL considers two SQL plans contextually equivalent when their observable behavior agrees for database instances satisfying the relevant portion of `Γ`.

The system lowers verified SQL histories to a typed relational representation, generates candidate reusable subprogram families, and obtains conservative equivalence evidence using constraint-aware formal/bounded reasoning where supported plus contract-preserving differential/counterexample execution elsewhere. The judgment is three-valued: `EQUIVALENT`, `NOT_EQUIVALENT`, or `UNKNOWN`; unknown pairs are not force-merged.

Library induction then searches for parameterized abstractions across equivalence-supported instances. Crucially, every learned abstraction stores a **scope contract** describing the sufficient conditions under which it may be reused. A new query can compose these operators only when the current warehouse/context satisfies their preconditions; otherwise generation falls back to the base Text2SQL method.

The research contribution is not SQL equivalence, e-graphs, program-library learning, execution validation, or memory individually. It is the claim that **context-dependent warehouse laws materially change which abstractions are discoverable/useful, and that encoding those laws as operator scope conditions enables safe compositional reuse**.

## Minimum Convincing Evidence

### Gate A — mechanism necessity

A controlled corpus of hard-positive and hard-negative SQL pairs/motif families must show that SemLibSQL-Γ moves the precision–recall frontier versus:

- lexical/token normalization;
- normalized AST clustering;
- Stitch-style syntax library learning;
- babble/LLMT-style learning with a strong fixed SQL equational theory;
- ReGAL-style execution-refactoring.

The critical result is not generic cluster quality. It is **better recovery of contextually equivalent cross-realization motifs without increasing false semantic merges**.

### Gate B — downstream value

On held-out motif compositions, SemLibSQL must outperform the strongest of:

- stateless Text2SQL;
- matched-context verified-query retrieval;
- AgentSM-like structured memory;
- GATE/curated semantic-grounding memory;
- generic learned library;
- manual/oracle semantic library as an upper-bound diagnostic.

### Safety gate

Hard negatives that violate a learned operator's applicability conditions must show that scoped use is safer than unscoped use.

## Kill Conditions

Kill the core paper thesis if any of the following holds:

1. babble/ReGAL/Stitch with generous SQL rewrite/equivalence support matches the proposed method;
2. improvements come only from alias/CTE/format normalization;
3. false merges materially increase when hard-positive recall improves;
4. verified-query/structured memory matches held-out composition performance at matched context;
5. a manual semantic library helps strongly while automatic induction recovers negligible benefit;
6. operator scope requires substantial manual business labeling, making “automatic induction” misleading.

Do **not** add RL, MCTS, multi-agent debate, extra verifiers, or more memory layers to rescue a failed Gate A.

## Experiment Design

- **Phase A corpus:** 8–12 semantic motifs; initial target 200–500 verified SQL programs; at least 3 warehouse schemas/projects; deliberately diverse SQL realization families.
- **Hard positives:** same motif, radically different SQL structure/dialect/decomposition.
- **Hard negatives:** highly similar SQL shape but decision-relevant semantic difference (grain, time cutoff, key, join cardinality, null policy, denominator, tie handling).
- **Phase B split:** individual motifs and some combinations observed in history; target combinations withheld.
- **Metrics:** hard-positive recall at fixed precision, false-merge rate, abstraction purity, coverage, semantic execution accuracy on held-out composition, first-attempt success, token/tool cost, negative-transfer rate.
- **Statistics:** paired task outcomes; bootstrap confidence intervals; McNemar for binary correctness where applicable; per-motif and per-database breakdowns.
- **Compute:** Gate A should be CPU/SQL-heavy and deliberately cheap; large-model training is not required to decide whether the core mechanism is worth continuing.

## Closest Baselines / Prior Work

| Work | What it establishes | Why it is not the claimed contribution |
|---|---|---|
| DreamCoder | learned reusable program libraries | not SQL/warehouse contextual equivalence |
| Stitch | scalable corpus-guided library learning | syntax/program corpus abstraction |
| babble / LLMT | library learning modulo an equational theory | strongest conceptual prior; theory is supplied and abstractions are not evaluated as warehouse-scoped Text2SQL concepts |
| ReGAL | reusable functions learned by execution-validated refactoring | no constraint-conditioned SQL equivalence/scope contract |
| U-semiring / VeriEQL | SQL equivalence under integrity constraints | no library induction/Text2SQL composition |
| AgentSM | structured reusable Text2SQL memory | retrieval/reuse, not new operator language induction |
| GATE | execution-grounded semantic memory | stores groundings, not abstraction + applicability-condition induction |
| Snowflake Semantic View Autopilot | workload/BI-driven semantic model generation | product-level semantic modeling; distinct evaluation object |

## Key Decisions

- Keep the contribution **one mechanism deep**: abstraction under contextual equivalence.
- Treat equivalence provers and e-graphs as components/baselines, not novelty claims.
- Preserve `UNKNOWN`; do not force equivalence for coverage.
- Make applicability scope a first-class artifact of every operator.
- Use a manual semantic library as an upper-bound diagnostic.
- Freeze all unrelated agent architecture until Gate A passes.

## Known Risks

- **Combination-of-known-techniques critique:** The paper must show an empirical phenomenon that generic LLMT/refactoring cannot reproduce.
- **Constraint quality:** Inferred warehouse contracts can be wrong; initial experiments should separate declared/high-confidence constraints from speculative learned ones.
- **Benchmark artificiality:** Hard positives/negatives must include realistic SQL idioms and not only hand-designed toy pairs.
- **Applicability leakage:** Test operators on near-miss contexts where one precondition is intentionally violated.

## Status

- [x] Idea selected
- [x] Literature landscape completed
- [x] Novelty risks identified
- [x] Internal adversarial review completed
- [x] Method and falsification plan frozen
- [ ] Formal independent reviewer acceptance
- [ ] Gate-A corpus instantiated/executed
- [ ] Gate-A mechanism result
- [ ] Gate-B held-out composition result
- [ ] Full method implementation
- [ ] Paper draft

## Next-Step Pointer

Proceed only to the experiment described in:

`research/text2sql-agent/refine-logs/EXPERIMENT_PLAN.md`

The first decision is **Gate A: does contract-conditioned equivalence change abstraction discovery beyond strong generic library-learning baselines?**
