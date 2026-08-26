# Fresh Idea Discovery Report — Text2SQL × Database/Data Agents

**Date:** 2026-08-24  
**Branch:** `research/text2sql-agent-fresh-2026-08-24`  
**Pipeline:** research-lit → batch idea generation → novelty screening → adversarial review → refinement  
**Independence rule:** started from `main`; this run does not use earlier Text2SQL idea rankings or conclusions.  
**Empirical pilots:** NOT RUN.  
**Formal external reviewer receipt:** UNAVAILABLE in this connector session; novelty/review assessments are provisional.

---

## Executive Summary

A fresh 2025–2026 literature pass shows that many formerly attractive Text2SQL ideas are now saturated: multi-turn RL, adaptive turn budgets, database probing, semantic memory, semantic IR/layers, candidate voting, execution verification, clarification, query repair, unknown-database routing, and AI-native SQL optimization all have close recent work.

The research process has now switched to an **anchor-paper failure-boundary workflow**. Raw novelty scores are secondary. Current priority is determined by whether a strong accepted system exposes a narrow, reproducible boundary that can be tested on 20–50 public/realistic examples before any new method is designed.

The active anchor set now spans two batches:

- **Interactive/agentic:** BIRD-INTERACT, SQL-Trail, DPC, VET, Spider 2.0/Spider2-DBT, MTSQL-R1.
- **SIGMOD/VLDB database systems:** ReSequel, OpenSQL, GLEAM, SemBench, SafeQL, SQL-Factory, and the VLDB-2026 large-schema subsetting analysis.

The current top reproduction probes are **WorldValid-MDD, SampledRewriteUnsoundness, SupervisionFanout-SQL, ErrorCarryover-SQL, Semantic Repeatability Gap, BudgetDistortion-BIRD, DownstreamImpact-GEM, FutureSnapshot-SQL, DenotationRewardTrap-SQL, and DocDrift-SQL**.

The common scientific pattern is **proxy failure**: accepted systems optimize a measurable proxy such as sample agreement, pairwise F1, schema recall, intermediate-label density, average semantic-query quality, or synthetic-corpus diversity, while the user ultimately cares about a different downstream property. No repair architecture is promoted until the proxy gap is empirically reproduced.

Detailed current artifacts:

- `idea-stage/IDEA_CANDIDATES.md` — reproduction-first shortlist.
- `idea-stage/ANCHOR_FAILURE_IDEAS_2026-08-26.md` — first anchor batch.
- `idea-stage/ANCHOR_FAILURE_IDEAS_DBVENUES_2026-08-26.md` — SIGMOD/VLDB anchor batch.
- `idea-stage/IDEA_POOL_EXPANSION_2026-08-26.md` — broader epistemic/system pool.

---

# 1. Fresh Literature Landscape

## 1.1 Autonomy is now a spectrum, not a binary

**Agentic-SQL Revisited** (arXiv:2608.15389) organizes Text2SQL along an autonomy axis: constrained, in-context, iterative, agentic, and reasoning-internalized. Its synthesis reports uneven transfer across Spider/BIRD/Spider2 and non-trivial costs from greater autonomy.

**SQL-Trail** and **MTSQL-R1** (ACL 2026) both train multi-turn database interaction. SQL-Trail uses interleaved execution feedback and adaptive turn budgets; MTSQL-R1 casts multi-turn Text2SQL as an MDP with propose→execute→verify→refine and persistent dialogue memory.

**BAP-SQL** (arXiv:2608.02876) makes observation planning budget-aware; gains are strongest under tight budgets and attenuate with larger budgets/models.

**AGENTIQL** already routes between a modular agentic pipeline and a simpler parser.

**Implication:** generic `route hard questions to an agent` or `adapt turns to difficulty` is crowded. A remaining question is whether the minimum qualitatively sufficient autonomy regime is a learnable counterfactual variable distinct from risk score/token budget.

---

## 1.2 Probing is crowded; observation representation is less studied

**SDE-SQL** performs self-driven SQL probes. **PV-SQL** probes concrete records to resolve value/column/join ambiguity. **PExA** uses parallel atomic SQL tests inspired by software testing. **BAP-SQL** decides how to form observations under budget.

Most of these systems optimize which action/probe to run. They usually still return rows/text to the LLM. That leaves a sharper interface question: for a given uncertainty, what is the smallest structured observation that preserves the evidence needed for the next decision?

Possible database-native evidence units include row count, distinct/key coverage, join match rate, null fraction, heavy hitters, quantiles, temporal bounds, many-to-many risk, uniqueness witnesses, and compact lineage/provenance fingerprints.

---

## 1.3 Candidate selection, verification, and repair are highly occupied

**R³-SQL** groups candidates by execution result, ranks result groups, and resamples when needed. **DPC** builds a Minimal Distinguishing Database and cross-checks SQL with Python/Pandas. **VET** executes stepwise semantics on the real database. **SafeQL** restricts refinement to a safe query space. Recent repair work also targets executable-but-wrong SQL.

**Implication:** multiple candidates + judge, execution reflection, counterexample DBs, and targeted clause repair should be baselines/components rather than paper novelty.

---

## 1.4 Text2SQL memory is now a mature sub-direction

**AgentSM** stores interpretable structured programs derived from execution traces. **Crystallization in Text-to-SQL** separates replay, retention, and held-out same-database transfer. **MERIT**, **SDAM**, **GATE**, and **EvoSQL** cover dynamic retrieval, contradiction-aware/evolving memories, execution-grounded semantic memory, and memory-augmented generator/critic loops.

**Implication:** basic success/failure memory, phase-aware retrieval, portable memory, or `store verified SQL` are no longer good standalone claims. More interesting are negative transfer under environmental change and causal propagation after a single incorrect history state.

---

## 1.5 Semantic planning/layers increasingly separate probabilistic grounding from deterministic execution

Recent semantic planning/layer work uses structured semantic representations and deterministic lowering. `semantic IR + compiler` is therefore crowded as a standalone contribution.

---

## 1.6 Unanswerability/refusal, benchmark repair and access control are crowded

Recent work covers answerability gating, structural abstention, benchmark annotation-error detection, gold audits, and RBAC-aware Text2SQL. Generic refusal, gold checking, and access-policy enforcement no longer suffice.

---

## 1.7 Database-venue anchors expose proxy failures

The SIGMOD/VLDB anchor batch adds a different class of boundaries:

- **ReSequel:** correctness is verified on sampled data, but users need semantic equivalence on the actual relevant database states.
- **OpenSQL:** one root `(question, SQL)` example becomes multiple intermediate targets, potentially multiplying annotation errors.
- **GLEAM:** pairwise entity-matching F1 may not align with downstream join/aggregate/rank correctness.
- **SemBench:** mean quality/cost may hide whether the user-visible decision changes across repeated stochastic executions.
- **Large-schema subsetting:** raw schema size may be less causal than ambiguity density or loss of low-salience relational bridges.
- **SafeQL:** DBMS feedback is rich for non-executable errors but silent on many executable semantic errors.
- **SQL-Factory:** structural/schema diversity may not measure coverage of the failure modes that downstream models actually need.

These are now the main source of new anchor-driven probes.

---

# 2. Directions Explicitly Killed as Standalone Novelty

Generic active schema/database probing; generic multi-turn RL; adaptive turn count/budget; planner/critic/verifier agent; multi-candidate voting/ranking; generic SQL repair; basic Text2SQL memory; semantic IR/compiler; generic clarification/EIG; minimal clause patching; unknown-database selection; generic benchmark error detection; generic no-answer refusal; generic dirty-key join agent; generic RBAC enforcement; generic transactional agent; generic constraint-based test database generation; generic SQL/Pandas cross-checking; generic entity matching; generic schema linking; generic query-equivalence checking; agentic GPU scheduling; and generic AI-SQL physical optimization are treated as prior art/baselines rather than new paper claims.

---

# 3. Current Research Queue

The canonical ranking is maintained in `idea-stage/IDEA_CANDIDATES.md`. The highest-priority probes deliberately mix Text2SQL and core data-system papers:

1. **WorldValid-MDD** — DPC counterexample-world realizability.
2. **SampledRewriteUnsoundness** — ReSequel sampled verification versus full-data semantics.
3. **SupervisionFanout-SQL** — OpenSQL bad-root label multiplication.
4. **ErrorCarryover-SQL** — long-horizon semantic-error half-life.
5. **Semantic Repeatability Gap** — SemBench decision stability.
6. **BudgetDistortion-BIRD** — BIRD-INTERACT budget-regime dependence.
7. **DownstreamImpact-GEM** — GLEAM F1 versus downstream analytical accuracy.
8. **FutureSnapshot-SQL** — natural data evolution as semantic test.
9. **DenotationRewardTrap-SQL** — false-positive execution rewards in agentic/RL systems.
10. **DocDrift-SQL** — real stale enterprise artifacts versus missing/current artifacts.

Each has an explicit kill condition in the detailed anchor documents.

---

# 4. Broader Idea Pool

The broader open-ended pool still contains potentially high-novelty directions such as CompletenessProof-SQL, AdaptiveInference-SQL/DataDredgeBench, ViewpointSafe-SQL, EvidenceSketch-SQL, Schema-Evolution × Memory Poisoning, MaximalSafeAnswer-SQL, DecisionMargin-SQL, SemanticJurisdiction-SQL, CausalGuard-SQL, QualityTaint-SQL, HarnessTransfer-SQL, and NonInvasive-SQL.

They are deliberately secondary to the anchor queue until a concrete failure phenomenon is established.

---

# 5. Hard Research Rules

1. **No method-first rescue.** If the anchor survives the boundary probe, the idea dies.
2. **No synthetic-only primary evidence when a public/real test is possible.** Synthetic stress tests are secondary.
3. **One failure variable at a time.** Do not bundle schema, memory, uncertainty, verification and planning.
4. **Simple baseline first.** If a deterministic rule or data cleanup explains the effect, kill the method claim.
5. **Old database mechanisms are prior art.** Adding an LLM is not enough.
6. **Artifact unavailable means BLOCKED.** Do not replace unreleased public data with a synthetic benchmark just to keep an idea alive.
7. **No formal PASS claims without reviewer receipts.** The independent cross-model reviewer backend is not available in this connector session.
8. **No empirical-result claims yet.** No training, SQL pilot or benchmark run has been performed in this session.

---

# 6. Next Boundary

The next step is no longer idea generation. It is the first 20–50-example reproduction probes, in roughly this cost order:

1. ReSequel full-data rewrite audit.
2. OpenSQL bad-gold augmentation audit.
3. SemBench repeated-decision stability audit.
4. DPC realizable-MDD audit.
5. MTSQL-R1 error-carryover intervention.
6. GLEAM F1-to-analytics correlation audit.
7. SafeQL executable-wrong versus DBMS-visible repair audit.
8. BIRD-INTERACT tight/free budget audit.

Until a boundary survives, designing a new architecture is intentionally blocked.
