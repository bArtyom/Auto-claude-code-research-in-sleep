# Fresh Text2SQL × Agent Idea Candidates

**Date:** 2026-08-24  
**Run:** independent from previous Text2SQL research branches  
**Formal reviewer:** unavailable in this connector session; novelty is provisional.  
**Pilots:** not run.

| # | Idea | Core hypothesis | Novelty | Feasibility | Status |
|---:|---|---|---:|---:|---|
| 1 | **EvidenceSketch-SQL** | task-adaptive DB evidence sketches beat raw probe rows at matched context/privacy budget | 7.5 | 8.5 | RECOMMENDED |
| 2 | **PoU-SQL** | checkable unanswerability certificates outperform abstention/classification for epistemic failures | 8.0 | 8.0 | RECOMMENDED |
| 3 | **GoldChallenge-SQL** | contestable, versioned gold oracles make training/evaluation more reliable than frozen labels | 7.5 | 8.5 | RECOMMENDED |
| 4 | **CausalGuard-SQL** | identifiability gating prevents descriptive SQL evidence from becoming unsupported causal claims | 7.5 | 7.0 | RECOMMENDED |
| 5 | **AutonomyOracle-SQL** | counterfactual minimal-autonomy labels improve cost/accuracy routing beyond difficulty or risk scores | 6.5–7 | 7.0 | RECOMMENDED WITH CAUTION |
| 6 | Schema-Evolution × Memory Poisoning | verified memories become harmful after controlled schema/business drift | 7.5 | 8.0 | STRONG BACKUP |
| 7 | CrossDB-Key Agent | explicit join-validity evidence improves heterogeneous cross-DB entity linkage | 7.0 | 7.5 | STRONG BACKUP |
| 8 | HarnessTransfer-SQL | orchestration strategies overfit benchmarks/harnesses independently of backbone quality | 7.0 | 6.5 | BACKUP |
| 9 | PolicyCounterfactual-SQL | closest authorized semantic relaxation is more useful than refusal-only policy handling | 7.0 | 6.5 | BACKUP |
| 10 | Model-Upgrade Memory Compatibility | legacy Text2SQL memory can show model-specific negative transfer after backbone upgrades | 6.5–7 | 8.0 | BACKUP |
| 11 | Stochastic AI-SQL Verifier | probabilistic AI-SQL needs statistical correctness/equivalence semantics | 6.5 | 7.0 | WATCH |
| 12 | AI-Function Necessity Compiler | choose whether probabilistic AI computation is needed before AI-query physical planning | 6.0 | 6.5 | WATCH |

## Five cheapest kill tests

1. **EvidenceSketch-SQL:** raw rows vs token-matched samples vs generic summaries vs fixed/adaptive sketches on 100–300 ambiguity cases.
2. **PoU-SQL:** remove one required fact/source from paired tasks; test missing-fact localization and recovery after restoration.
3. **GoldChallenge-SQL:** inject realistic gold corruption and test whether disagreement-triggered auditing catches it without exonerating bad model outputs.
4. **AutonomyOracle-SQL:** force the same tasks through several autonomy regimes and test whether the cheapest-successful regime contains signal beyond difficulty/budget.
5. **CausalGuard-SQL:** SCM-backed relational tasks with matched observational evidence but different causal structures; measure unsupported causal-answer reduction.

## Directions intentionally killed as standalone ideas

Generic active probing, generic multi-turn RL, adaptive turn count, planner/critic/verifier loops, multi-candidate voting, basic Text2SQL memory, semantic IR/compiler, EIG clarification, minimal clause patching, unknown-database routing, claim provenance, agentic GPU scheduling, and generic AI-SQL physical optimization are too close to current work.

## Canonical report

See `research/text2sql-agent-fresh-2026-08-24/idea-stage/IDEA_REPORT.md` for the literature map, 36 raw ideas, collision analysis, internal adversarial review, and experiment kill gates.