# Fresh Text2SQL × Agent Idea Candidates

**Updated:** 2026-08-26  
**Run:** independent from previous Text2SQL research branches  
**Formal reviewer:** unavailable in this connector session; novelty is provisional.  
**Pilots:** not run.  
**Current process:** anchor paper → failure boundary → 20–50 example reproduction/kill test → only then method design.

## Anchor-driven reproduction shortlist — highest priority

These ideas are prioritized because each begins from a specific strong public paper/benchmark and one falsifiable boundary, rather than from a preferred mechanism.

| # | Idea | Anchor | Failure boundary | First test | Status |
|---:|---|---|---|---|---|
| 1 | **WorldValid-MDD** | DPC, ACL 2026 | distinguishing DB may violate real integrity/business constraints | audit/regenerate 30–50 MDDs with DDL→dbt/domain constraints | PRIORITY REPRODUCTION |
| 2 | **SampledRewriteUnsoundness** | ReSequel, VLDB 2026 | rewrite accepted on sampled data may be semantically wrong on full/rare data | replay 30–50 accepted rewrites on full DB + rare/boundary slices | PRIORITY REPRODUCTION |
| 3 | **SupervisionFanout-SQL** | OpenSQL, PVLDB 2026 | one bad gold pair may fan out into several correlated wrong intermediate labels | run 20–50 audited bad BIRD roots through augmentation, no training | PRIORITY REPRODUCTION |
| 4 | **ErrorCarryover-SQL** | MTSQL-R1, ACL 2026 | persistent history may amplify one early plausible semantic error | inject one controlled error into 30–50 dialogues | PRIORITY REPRODUCTION |
| 5 | **Semantic Repeatability Gap** | SemBench, VLDB 2026 | good mean quality may hide user-visible decision flips across repeated semantic queries | repeat 20–50 queries and score top-k/rank/set stability | PRIORITY REPRODUCTION |
| 6 | **BudgetDistortion-BIRD** | BIRD-INTERACT, ICLR 2026 Oral | strict interaction budget may change policy/ranking | rerun 30–50 tasks under tight/medium/free-ish budgets | PRIORITY REPRODUCTION |
| 7 | **DownstreamImpact-GEM** | GLEAM, SIGMOD 2026 | pairwise entity-matching F1 may select the wrong system for downstream analytics | correlate F1 with aggregate/rank error across thresholds/baselines | PRIORITY REPRODUCTION |
| 8 | **FutureSnapshot-SQL** | execution-guided/RL Text2SQL + Spider semantic eval | static-snapshot correctness may hide future-data semantic errors | evaluate SQL from snapshot t on natural t+1/t+2 data | PRIORITY REPRODUCTION |
| 9 | **DenotationRewardTrap-SQL** | SQL-Trail / execution-RL | execution reward may reinforce false-positive denotation shortcuts | re-score 20–50 successes with test suites/distinguishing worlds | PRIORITY REPRODUCTION |
| 10 | **DocDrift-SQL** | Spider 2.0 | stale but plausible enterprise artifacts may be worse than missing docs | mine real repo/benchmark history before any synthetic perturbation | PRIORITY REPRODUCTION |
| 11 | **MDD-Stability-SQL** | DPC, ACL 2026 | verifier decision may depend on arbitrary choice of counterexample DB | 5–10 valid MDDs per candidate duel | STRONG |
| 12 | **RepoLineageDepth-SQL** | Spider2-DBT | failure may cliff at dependency/macro depth rather than raw schema size | structural analysis of all 68 DBT tasks | STRONG / VERY CHEAP |
| 13 | **Silent-Semantic Boundary of SafeQL** | SafeQL, VLDB 2026 | DBMS-guided repair may fail exactly where SQL executes but meaning is wrong | stratify 30–50 errors into DBMS-visible vs executable-wrong | STRONG |
| 14 | **Schema-Ambiguity Phase Transition** | VLDB 2026 schema-subsetting EA&B | subsetting benefit may depend on ambiguity/decoy density rather than raw schema size | natural-schema regression when BigBird/SKALPEL artifacts release | STRONG — ARTIFACT BLOCKED |
| 15 | **Feedback-Information Boundary** | SQL-Trail / ReEx-SQL | feedback type may matter more than nominal difficulty | label feedback type and correction probability | STRONG |
| 16 | **SimulatorTransfer-BIRD** | BIRD-INTERACT | interaction policy may specialize to simulator realization | same symbolic facts, alternative response realizations | STRONG |
| 17 | **Operator-Composition Cliff** | SemBench, VLDB 2026 | semantic-operator errors may compose nonlinearly | relate query quality to existing operator sequence/depth | STRONG |
| 18 | **Feedback-Skew Adaptivity Trap** | GLEAM, SIGMOD 2026 | online matching feedback may improve hot segments while harming cold ones | skew feedback order and evaluate untouched segments | STRONG |
| 19 | **DiversityUtility-SQL** | SQL-Factory, PVLDB/VLDB 2026 | structural diversity may not predict coverage of real model failure modes | compare generated-corpus taxonomy with held-out error taxonomy | STRONG DIAGNOSTIC |
| 20 | **JoinBridge Blind Spot** | VLDB 2026 schema-subsetting EA&B | pruning may disproportionately drop relational bridge context | failure audit after artifact release | STRONG — ARTIFACT BLOCKED |

### Anchor-driven kill rules

- If the anchor system survives the boundary probe, **kill the derived idea**.
- If a deterministic/simple baseline explains the effect, kill the method claim; keep only an analysis/benchmark result if scientifically useful.
- If the phenomenon appears only after fabricated synthetic perturbations, do not use it as the primary paper claim.
- Do not add a repair architecture until the failure is reproduced on public/realistic evidence.
- If a required public artifact is not released, mark the test **BLOCKED** rather than replacing it with a synthetic substitute.
- Old mechanisms such as constraint-based DB generation, query-equivalence checking, test-suite evaluation, SQL/Pandas cross-checking, entity resolution, or schema linking are baselines; novelty must be the newly reproduced failure mechanism/evaluation object.

## Why the second anchor batch changed the queue

The SIGMOD/VLDB papers exposed several unusually cheap tests where the **paper's optimization proxy may differ from the user's real objective**:

- ReSequel verifies rewrite correctness on sampled data; the user needs full semantic equivalence.
- OpenSQL multiplies one `(question, SQL)` pair into several intermediate targets; a bad root label may therefore contaminate multiple modules.
- GLEAM optimizes pairwise entity-matching F1; an analyst cares about the accuracy of downstream joins/aggregates/rankings.
- SemBench reports result quality/cost and supports repeated runs; a user may care whether the final decision changes identity between executions.
- Schema subsetting optimizes context/recall trade-offs; the catastrophic missing information may specifically be low-salience relational bridges.
- SQL-Factory optimizes structural/schema diversity; downstream learning value may instead be concentrated in semantic boundary cases matching real model errors.

This queue therefore favors **proxy-failure audits** that can be falsified before any new architecture is built.

## Broader provisional idea pool

The following pool remains useful for exploration, but it is secondary to anchor-driven reproduction until one of its failure boundaries is empirically verified.

| # | Idea | Core hypothesis | Novelty | Feasibility | Status |
|---:|---|---|---:|---:|---|
| 1 | **CompletenessProof-SQL** | correct SQL is insufficient; agents need checkable answer-completeness/scope certificates over partially complete data | 8.5 | 8.0 | DEEP NOVELTY NEEDED |
| 2 | **AdaptiveInference-SQL / DataDredgeBench** | adaptive SQL exploration can create false discoveries; episode-level validity protocols can control them | 8.0–8.5 | 8.0 | DEEP NOVELTY NEEDED |
| 3 | **ViewpointSafe-SQL** | policy-compliant/RLS-filtered SQL may not justify the global claim; agent must prove authorized-view sufficiency | 8.0 | 8.5 | DEEP NOVELTY NEEDED |
| 4 | **EvidenceSketch-SQL** | task-adaptive DB evidence sketches beat raw probe rows at matched context/privacy budget | 7.5 | 8.5 | RECOMMENDED |
| 5 | **Schema-Evolution × Memory Poisoning** | verified memories become harmful after controlled schema/business drift | 7.5 | 8.0 | RECOMMENDED |
| 6 | **MaximalSafeAnswer-SQL** | when a global answer is incomplete, return the largest useful scope that can be certified complete | 7.5–8.0 | 7.5 | STRONG BET |
| 7 | **DecisionMargin-SQL** | a decision/ranking can be provably stable even when the exact number is uncertain under missing data | 7.5–8.0 | 7.0 | STRONG BET |
| 8 | **SemanticJurisdiction-SQL** | business definitions are legitimately scoped by team/task/time; retrieval should preserve jurisdictional polysemy | 7.5 | 8.0 | STRONG BET |
| 9 | **CausalGuard-SQL** | identifiability gating prevents descriptive SQL evidence from becoming unsupported causal claims | 7.0–7.5 | 7.0 | RECOMMENDED |
| 10 | **QualityTaint-SQL** | failed freshness/uniqueness/completeness tests should propagate through lineage into what claims are allowed | 7.0–7.5 | 8.5 | STRONG BET |
| 11 | **HarnessTransfer-SQL** | orchestration strategies overfit benchmarks/harnesses independently of backbone quality | 7.0 | 6.5 | RECOMMENDED |
| 12 | **NonInvasive-SQL** | exploration should optimize information gain against production-workload interference, not only token cost | 7.0–7.5 | 7.0 | STRONG BET |
| 13 | **SourceAuthority-Agent** | heterogeneous evidence selection should optimize authority/validity, not only semantic relevance | 7.0–7.5 | 8.0 | BACKUP |
| 14 | **MeasurementType-SQL** | nominal/ordinal/interval/ratio/unit/time semantics catch executable-but-invalid analytical operations | 7.0 | 8.0 | BACKUP |
| 15 | **AutonomyOracle-SQL** | counterfactual minimal-autonomy labels improve cost/accuracy routing beyond difficulty or risk scores | 6.5–7.0 | 7.0 | CAUTION |
| 16 | **PoU-SQL** | checkable missing-information certificates add value beyond structural/latent abstention | 6.5–7.0 | 8.0 | BACKUP — NARROWED |
| 17 | **TemporalCoherence-Agent** | multi-source evidence should describe one coherent as-of world | 6.5–7.0 | 7.5 | BACKUP — TRANSACTION COLLISION |
| 18 | **Model-Upgrade Memory Compatibility** | legacy Text2SQL memory can show model-specific negative transfer after backbone upgrades | 6.5–7.0 | 8.0 | BACKUP |
| 19 | **Privilege-Scoped Memory** | semantic memory learned under one DB privilege context can leak or mislead under another | 6.5–7.0 | 8.0 | BACKUP — SECURITY COLLISION |
| 20 | **GoldChallenge-SQL** | contestable, versioned gold oracles improve on existing annotation-error detection/auditing | 6.0–6.5 | 8.0 | BACKUP — NARROWED |

## Research-process rule

The main risk is not lack of architectural creativity. It is failing to establish that a strong existing system has a real, narrow, reproducible boundary. Anchor-driven probes therefore outrank raw novelty scores until a failure phenomenon is reproduced.

The current anchor set spans BIRD-INTERACT, SQL-Trail, DPC, VET, Spider 2.0/Spider2-DBT, MTSQL-R1, ReSequel, OpenSQL, GLEAM, SemBench, SafeQL, SQL-Factory, and the VLDB-2026 schema-subsetting analysis.

## Existing directions intentionally kept killed as standalone ideas

Generic active probing, generic multi-turn RL, adaptive turn count, planner/critic/verifier loops, multi-candidate voting, basic Text2SQL memory, semantic IR/compiler, EIG clarification, minimal clause patching, unknown-database routing, generic claim provenance, agentic GPU scheduling, generic benchmark-error detection, generic unanswerability refusal, generic dirty-key join agents, generic RBAC enforcement, generic transactional agents, generic constraint-based DB generation, generic SQL/Pandas cross-checking, generic entity matching, generic schema linking, generic query-equivalence checking, and generic AI-SQL physical optimization remain too close to current work.

## Detailed artifacts

- `research/text2sql-agent-fresh-2026-08-24/idea-stage/ANCHOR_FAILURE_IDEAS_2026-08-26.md` — first anchor batch: six evidence cards, 18 detailed failure-derived ideas, 25 compact secondary ideas.
- `research/text2sql-agent-fresh-2026-08-24/idea-stage/ANCHOR_FAILURE_IDEAS_DBVENUES_2026-08-26.md` — SIGMOD/VLDB anchor batch: seven evidence cards, 20 detailed failure-derived ideas, 20 compact probes.
- `research/text2sql-agent-fresh-2026-08-24/idea-stage/IDEA_POOL_EXPANSION_2026-08-26.md` — broader 50-candidate epistemic/system pool.
- `research/text2sql-agent-fresh-2026-08-24/idea-stage/IDEA_REPORT.md` — initial fresh literature map and saturated-direction map.
