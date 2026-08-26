# Fresh Text2SQL × Agent Idea Candidates

**Updated:** 2026-08-26  
**Run:** independent from previous Text2SQL research branches  
**Formal reviewer:** unavailable in this connector session; novelty is provisional.  
**Pilots:** not run.  
**Latest expansion:** adds epistemic completeness, adaptive statistical validity, access-control epistemics, answer robustness, data-quality semantics, and database-workload externalities.

## Combined provisional shortlist

| # | Idea | Core hypothesis | Novelty | Feasibility | Status |
|---:|---|---|---:|---:|---|
| 1 | **CompletenessProof-SQL** | correct SQL is insufficient; agents need checkable answer-completeness/scope certificates over partially complete data | 8.5 | 8.0 | NEW TOP BET — DEEP NOVELTY NEEDED |
| 2 | **AdaptiveInference-SQL / DataDredgeBench** | adaptive SQL exploration can create false discoveries; episode-level validity protocols can control them | 8.0–8.5 | 8.0 | NEW TOP BET — DEEP NOVELTY NEEDED |
| 3 | **ViewpointSafe-SQL** | policy-compliant/RLS-filtered SQL may not justify the global claim; agent must prove authorized-view sufficiency | 8.0 | 8.5 | NEW TOP BET — DEEP NOVELTY NEEDED |
| 4 | **EvidenceSketch-SQL** | task-adaptive DB evidence sketches beat raw probe rows at matched context/privacy budget | 7.5 | 8.5 | RECOMMENDED |
| 5 | **Schema-Evolution × Memory Poisoning** | verified memories become harmful after controlled schema/business drift | 7.5 | 8.0 | RECOMMENDED |
| 6 | **MaximalSafeAnswer-SQL** | when a global answer is incomplete, return the largest useful scope that can be certified complete | 7.5–8.0 | 7.5 | NEW STRONG BET |
| 7 | **DecisionMargin-SQL** | a decision/ranking can be provably stable even when the exact number is uncertain under missing data | 7.5–8.0 | 7.0 | NEW STRONG BET |
| 8 | **SemanticJurisdiction-SQL** | business definitions are legitimately scoped by team/task/time; retrieval should preserve jurisdictional polysemy | 7.5 | 8.0 | NEW STRONG BET |
| 9 | **CausalGuard-SQL** | identifiability gating prevents descriptive SQL evidence from becoming unsupported causal claims | 7.0–7.5 | 7.0 | RECOMMENDED |
| 10 | **QualityTaint-SQL** | failed freshness/uniqueness/completeness tests should propagate through lineage into what claims are allowed | 7.0–7.5 | 8.5 | NEW STRONG BET |
| 11 | **HarnessTransfer-SQL** | orchestration strategies overfit benchmarks/harnesses independently of backbone quality | 7.0 | 6.5 | RECOMMENDED |
| 12 | **NonInvasive-SQL** | exploration should optimize information gain against production-workload interference, not only token cost | 7.0–7.5 | 7.0 | NEW STRONG BET |
| 13 | **SourceAuthority-Agent** | heterogeneous evidence selection should optimize authority/validity, not only semantic relevance | 7.0–7.5 | 8.0 | NEW BACKUP |
| 14 | **MeasurementType-SQL** | nominal/ordinal/interval/ratio/unit/time semantics catch executable-but-invalid analytical operations | 7.0 | 8.0 | NEW BACKUP |
| 15 | **AutonomyOracle-SQL** | counterfactual minimal-autonomy labels improve cost/accuracy routing beyond difficulty or risk scores | 6.5–7.0 | 7.0 | RECOMMENDED WITH CAUTION |
| 16 | **PoU-SQL** | checkable missing-information certificates add value beyond structural/latent abstention | 6.5–7.0 | 8.0 | BACKUP — NARROWED |
| 17 | **TemporalCoherence-Agent** | multi-source evidence should describe one coherent as-of world | 6.5–7.0 | 7.5 | BACKUP — TRANSACTION COLLISION |
| 18 | **Model-Upgrade Memory Compatibility** | legacy Text2SQL memory can show model-specific negative transfer after backbone upgrades | 6.5–7.0 | 8.0 | BACKUP |
| 19 | **Privilege-Scoped Memory** | semantic memory learned under one DB privilege context can leak or mislead under another | 6.5–7.0 | 8.0 | BACKUP — SECURITY COLLISION |
| 20 | **GoldChallenge-SQL** | contestable, versioned gold oracles improve on existing annotation-error detection/auditing | 6.0–6.5 | 8.0 | BACKUP — NARROWED |

## Why the pool changed

The new literature pass exposed a deeper frontier than ordinary SQL generation:

- Classical incomplete-database theory shows that **query correctness and answer completeness are different properties**. Current Text2SQL evaluation almost always assumes the observed DB is the world.
- July-2026 RBAC Text2SQL work makes generic policy compliance crowded, but RLS silently filtering rows creates a separate failure: **a legal query can still support an unjustified global claim**.
- Classical adaptive-data-analysis theory plus 2026 LLM p-hacking work implies that autonomous analytics agents can create statistically invalid findings simply by adaptively exploring many SQL specifications.
- Agentic Transaction/Cordon now crowd generic `ACID for agents`; any temporal idea must focus on heterogeneous read-only evidence coherence rather than generic commit/rollback.
- Modern catalogs/dbt expose freshness, lineage and data-quality tests, so `give quality metadata to the LLM` is not enough; the stronger question is whether failed guarantees **taint specific final claims**.

## Immediate deep-novelty queue

Before any empirical pilot, perform focused literature checks in this order:

1. CompletenessProof-SQL — query completeness / partial closed world / certain-best answers × Text2SQL.
2. AdaptiveInference-SQL — adaptive data analysis / reusable holdout / anytime-valid inference × autonomous analytics agents.
3. ViewpointSafe-SQL — RBAC full paper + inference control / authorized-view completeness.
4. DecisionMargin-SQL — robust query processing / database causality / answer sensitivity / incomplete-data decisions.
5. SemanticJurisdiction-SQL — contextual ontologies / metric stores / multi-stakeholder enterprise semantics.
6. QualityTaint-SQL — data contracts / quality propagation / lineage / claim-level reliability.
7. NonInvasive-SQL — production workload management / AQP / replicas / cost-aware exploration.
8. MeasurementType-SQL — dimensional analysis / measurement scales / typed relational algebra.

## Existing directions intentionally kept killed as standalone ideas

Generic active probing, generic multi-turn RL, adaptive turn count, planner/critic/verifier loops, multi-candidate voting, basic Text2SQL memory, semantic IR/compiler, EIG clarification, minimal clause patching, unknown-database routing, generic claim provenance, agentic GPU scheduling, generic benchmark-error detection, generic unanswerability refusal, generic dirty-key join agents, generic RBAC enforcement, generic transactional agents, and generic AI-SQL physical optimization remain too close to current work.

## Detailed expansion

See `research/text2sql-agent-fresh-2026-08-24/idea-stage/IDEA_POOL_EXPANSION_2026-08-26.md` for 50 new candidates, literature collisions, twelve detailed idea cards, and falsification conditions.