# Fresh Text2SQL × Agent Idea Candidates

**Date:** 2026-08-24  
**Run:** independent from previous Text2SQL research branches  
**Formal reviewer:** unavailable in this connector session; novelty is provisional.  
**Pilots:** not run.  
**Latest novelty recheck:** includes August 2026 work found after the first-pass ranking.

| # | Idea | Core hypothesis | Novelty | Feasibility | Status |
|---:|---|---|---:|---:|---|
| 1 | **EvidenceSketch-SQL** | task-adaptive DB evidence sketches beat raw probe rows at matched context/privacy budget | 7.5 | 8.5 | RECOMMENDED |
| 2 | **Schema-Evolution × Memory Poisoning** | verified memories become harmful after controlled schema/business drift | 7.5 | 8.0 | RECOMMENDED |
| 3 | **CausalGuard-SQL** | identifiability gating prevents descriptive SQL evidence from becoming unsupported causal claims | 7.0–7.5 | 7.0 | RECOMMENDED |
| 4 | **HarnessTransfer-SQL** | orchestration strategies overfit benchmarks/harnesses independently of backbone quality | 7.0 | 6.5 | RECOMMENDED |
| 5 | **AutonomyOracle-SQL** | counterfactual minimal-autonomy labels improve cost/accuracy routing beyond difficulty or risk scores | 6.5–7 | 7.0 | RECOMMENDED WITH CAUTION |
| 6 | **PoU-SQL** | checkable missing-information certificates add value beyond structural/latent abstention | 6.5–7 | 8.0 | STRONG BACKUP — NARROWED |
| 7 | **Model-Upgrade Memory Compatibility** | legacy Text2SQL memory can show model-specific negative transfer after backbone upgrades | 6.5–7 | 8.0 | STRONG BACKUP |
| 8 | **GoldChallenge-SQL** | contestable, versioned gold oracles improve on existing annotation-error detection/auditing | 6.0–6.5 | 8.0 | BACKUP — NARROWED |
| 9 | PolicyCounterfactual-SQL | closest authorized semantic relaxation is more useful than refusal-only policy handling | 7.0 | 6.5 | BACKUP |
| 10 | CrossDB-Key Audit/Benchmark | explicitly evaluate join-validity evidence for heterogeneous dirty-key integration | 5.5–6.0 | 7.5 | BENCHMARK-ONLY CANDIDATE |
| 11 | Stochastic AI-SQL Verifier | probabilistic AI-SQL needs statistical correctness/equivalence semantics | 6.5 | 7.0 | WATCH |
| 12 | AI-Function Necessity Compiler | choose whether probabilistic AI computation is needed before AI-query physical planning | 6.0 | 6.5 | WATCH |

## Why the ranking changed after the second novelty pass

- **PoU-SQL moved down:** `Never the Number` (2026-08-14) proposes structural abstention for database/enterprise AI systems, and ACL 2026 `LatentRefusal` directly tackles answerability gating. A viable PoU paper must therefore contribute a *machine-checkable minimal-missing-information certificate* and recovery test, not merely refusal/answerability detection.
- **GoldChallenge-SQL moved down:** `SAR-Agent` already detects Text2SQL benchmark annotation errors through database interaction, and ACL 2026 `GBV-SQL` includes a Gold Error audit/typology. The only defensible delta is an *online contestable-oracle protocol* with versioned confidence and training/leaderboard integration.
- **CrossDB-Key moved down:** DAB already makes ill-formatted cross-database join keys a core benchmark challenge, and production systems publicly describe deterministic cross-database join inference. The safer research angle is a focused audit/benchmark of evidence requirements rather than claiming a generic new join agent.
- **Schema-Evolution × Memory Poisoning moved up:** EvoSchema studies Text2SQL under schema evolution, while AgentSM/Crystallization/GATE study reusable memory, but the failure of previously verified memory after controlled schema/business evolution remains a clean intersection with strong falsifiability.
- **HarnessTransfer-SQL moved up:** Agentic-SQL Revisited reports uneven cross-benchmark transfer and DataSpace shows large harness effects; directly measuring policy/harness overfitting has a clear empirical object and does not require inventing another agent module.

## Five cheapest kill tests

1. **EvidenceSketch-SQL:** raw rows vs token-matched samples vs generic summaries vs fixed/adaptive sketches on 100–300 ambiguity cases.
2. **Schema-Evolution × Memory Poisoning:** first establish beneficial memory, apply EvoSchema-style changes, then measure stale-memory negative transfer and whether trivial schema-version tagging solves it.
3. **CausalGuard-SQL:** SCM-backed relational tasks with matched observational evidence but different causal structures; measure unsupported causal-answer reduction and false refusals.
4. **HarnessTransfer-SQL:** hold backbone fixed and test several canonical orchestration policies across BIRD/Spider2/AIFunc/data-agent task families; measure policy ranking instability.
5. **AutonomyOracle-SQL:** force the same tasks through several autonomy regimes and test whether the cheapest-successful regime contains signal beyond difficulty/risk/budget.

## Directions intentionally killed as standalone ideas

Generic active probing, generic multi-turn RL, adaptive turn count, planner/critic/verifier loops, multi-candidate voting, basic Text2SQL memory, semantic IR/compiler, EIG clarification, minimal clause patching, unknown-database routing, claim provenance, agentic GPU scheduling, generic benchmark-error detection, generic unanswerability refusal, generic dirty-key join agents, and generic AI-SQL physical optimization are too close to current work.

## Canonical report

See `research/text2sql-agent-fresh-2026-08-24/idea-stage/IDEA_REPORT.md` for the literature map, 36 raw ideas, collision analysis, internal adversarial review, and experiment kill gates. The compact ranking in this file supersedes the report's first-pass ranking where they differ.