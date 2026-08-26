# Fresh Text2SQL × Agent Idea Candidates

**Updated:** 2026-08-26  
**Run:** independent from previous Text2SQL research branches  
**Formal reviewer:** unavailable in this connector session; novelty is provisional.  
**Pilots:** not run.  
**Latest process change:** anchor paper → failure boundary → 20–50 example reproduction/kill test → only then method design.

## Anchor-driven reproduction shortlist — highest priority

These ideas are prioritized for the next stage because each begins from a specific strong public paper/benchmark and one falsifiable boundary, rather than from a preferred mechanism.

| # | Idea | Anchor | Failure boundary | First test | Status |
|---:|---|---|---|---|---|
| 1 | **WorldValid-MDD** | DPC, ACL 2026 | distinguishing DB may violate real integrity/business constraints | audit/regenerate 30–50 MDDs with DDL→dbt/domain constraints | PRIORITY REPRODUCTION |
| 2 | **ErrorCarryover-SQL** | MTSQL-R1, ACL 2026 | persistent history may amplify one early plausible semantic error | inject one controlled error into 30–50 dialogues | PRIORITY REPRODUCTION |
| 3 | **BudgetDistortion-BIRD** | BIRD-INTERACT, ICLR 2026 Oral | strict interaction budget may change policy/ranking | rerun 30–50 tasks under tight/medium/free-ish budgets | PRIORITY REPRODUCTION |
| 4 | **FutureSnapshot-SQL** | execution-guided/RL Text2SQL + Spider semantic eval | static-snapshot correctness may hide future-data semantic errors | evaluate SQL from snapshot t on natural t+1/t+2 data | PRIORITY REPRODUCTION |
| 5 | **DenotationRewardTrap-SQL** | SQL-Trail / execution-RL | execution reward may reinforce false-positive denotation shortcuts | re-score 20–50 successes with test suites/distinguishing worlds | PRIORITY REPRODUCTION |
| 6 | **DocDrift-SQL** | Spider 2.0 | stale but plausible enterprise artifacts may be worse than missing docs | mine real repo/benchmark history before any synthetic perturbation | PRIORITY REPRODUCTION |
| 7 | **MDD-Stability-SQL** | DPC, ACL 2026 | verifier decision may depend on arbitrary choice of counterexample DB | 5–10 valid MDDs per candidate duel | STRONG |
| 8 | **RepoLineageDepth-SQL** | Spider2-DBT | failure may cliff at dependency/macro depth rather than raw schema size | structural analysis of all 68 DBT tasks | STRONG / VERY CHEAP |
| 9 | **Feedback-Information Boundary** | SQL-Trail / ReEx-SQL | feedback type may matter more than nominal difficulty | label feedback type and correction probability | STRONG |
| 10 | **SimulatorTransfer-BIRD** | BIRD-INTERACT | interaction policy may specialize to simulator realization | same symbolic facts, alternative response realizations | STRONG |

### Anchor-driven kill rules

- If the anchor system survives the boundary probe, **kill the derived idea**.
- If a deterministic/simple baseline explains the effect, kill the method claim; keep only an analysis/benchmark result if scientifically useful.
- If the phenomenon appears only after fabricated synthetic perturbations, do not use it as the primary paper claim.
- Do not add a repair architecture until the failure is reproduced on public/realistic evidence.
- Prior-art guardrails matter: constraint-based test generation, test-suite semantic evaluation, and SQL/Pandas semantic mismatch already exist; novelty must lie in the new failure mechanism or evaluation object.

## Broader provisional idea pool

The following pool remains useful for exploration, but it is **secondary to the anchor-driven reproduction shortlist** until one of its failure boundaries is empirically verified.

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

## Why the research process changed

The main risk in Text2SQL ideation is not lack of architectural creativity. It is failing to establish that a strong existing system has a real, narrow, reproducible boundary. The anchor-driven queue therefore outranks raw novelty scores until a failure phenomenon is reproduced.

The strongest current anchors are BIRD-INTERACT, SQL-Trail, DPC, VET, Spider 2.0/Spider2-DBT, and MTSQL-R1 because they provide a strong accepted result, relatively clear experimental protocol, public data/code for most cases, and specific boundaries that can be tested cheaply.

## Existing directions intentionally kept killed as standalone ideas

Generic active probing, generic multi-turn RL, adaptive turn count, planner/critic/verifier loops, multi-candidate voting, basic Text2SQL memory, semantic IR/compiler, EIG clarification, minimal clause patching, unknown-database routing, generic claim provenance, agentic GPU scheduling, generic benchmark-error detection, generic unanswerability refusal, generic dirty-key join agents, generic RBAC enforcement, generic transactional agents, generic constraint-based DB generation, generic SQL/Pandas cross-checking, and generic AI-SQL physical optimization remain too close to current work.

## Detailed artifacts

- `research/text2sql-agent-fresh-2026-08-24/idea-stage/ANCHOR_FAILURE_IDEAS_2026-08-26.md` — evidence cards for six anchors, 18 detailed failure-derived ideas, 25 compact secondary ideas, and kill conditions.
- `research/text2sql-agent-fresh-2026-08-24/idea-stage/IDEA_POOL_EXPANSION_2026-08-26.md` — broader 50-idea epistemic/system expansion.
- `research/text2sql-agent-fresh-2026-08-24/idea-stage/IDEA_REPORT.md` — initial fresh literature map and first-pass pool.
