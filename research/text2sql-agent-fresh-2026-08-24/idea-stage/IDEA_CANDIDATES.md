# Fresh Text2SQL × Agent Idea Candidates

**Updated:** 2026-08-27  
**Run:** independent fresh Text2SQL research branch  
**Current gate:** strong anchor → reproducible limitation → explicit algorithm delta → one primary quantitative metric → 20–50 example kill test.  
**Empirical pilots:** NOT RUN.  
**Formal independent reviewer:** UNAVAILABLE in this connector session; novelty remains provisional.

## Method-only shortlist — current priority

The previous pool intentionally included analysis/benchmark ideas. Under the stricter 2026-08-27 rule, only candidates with an explicit method change, fixed input/intermediate/output definition, strong baseline, primary metric and falsifiable low-cost test enter the active pool.

| Rank | Method idea | Main anchor | Exact method object | Primary metric | First kill test | Status |
|---:|---|---|---|---|---|---|
| 1 | **Constraint-Conditioned MDD SQL Selector** | DPC, ACL 2026 | require DPC distinguishing worlds to satisfy trusted target constraints; allow UNRESOLVED | Candidate Selection Top-1 EX on constraint-sensitive subset | 40 DPC candidate duels | ACTIVE METHOD CANDIDATE |
| 2 | **Risk-Adaptive Execution Checkpointing** | VET, Findings ACL 2026 | learn per-step causal checkpoint criticality under a hard DB-call budget | EX@50% intermediate DB-call budget | 50 BIRD Moderate/Challenging traces | ACTIVE METHOD CANDIDATE |
| 3 | **Contrastive Value-Link Probe Selector** | DIVER, SIGMOD 2026 | choose DIVER probes by information gain over explicit value-link hypotheses | Value Linking F1@≤3 probes | 40 ambiguous BIRD value-link cases | ACTIVE METHOD CANDIDATE |
| 4 | **Error-Targeted SQL Synthesis Scheduler** | SQL-Factory, PVLDB 19(3) | allocate a fixed synthesis budget from the target student's residual error signatures | Held-out DB EX under equal synthetic budget G | 40 held-out questions + 2K-pair pilot | ACTIVE METHOD CANDIDATE |
| 5 | **Predicted-History Curriculum Training** | MTSQL-R1, ACL 2026 | train on naturally model-generated executable-wrong dialogue histories without changing memory architecture | Predicted-History EX | 40 natural error-carryover turns | ACTIVE, HIGHER INCREMENTAL RISK |

## Why these five survived

1. **DPC:** the paper explicitly acknowledges that synthesized environments can diverge from real distributions under implicit constraints. The idea changes what evidence is admissible to the selector; it is not merely a constraint-solver add-on.
2. **VET:** the paper explicitly identifies per-step DB waiting as a latency limitation. The idea optimizes checkpoint necessity *inside* the existing trace rather than introducing a generic query router.
3. **DIVER:** generic probing is crowded, but DIVER's own tool-route ablation does not improve EX; the proposed method optimizes an explicit value-link hypothesis-discrimination objective and is judged by F1 under a fixed probe budget.
4. **SQL-Factory:** static structural curriculum already exists (e.g. SAC-SQL), so the surviving delta is closed-loop generation allocation from a particular student's measured residual errors under equal generation/training budgets.
5. **MTSQL-R1:** structured-history work such as Rose-SQL already exists, so the surviving delta is deployment-faithful training-state distribution over naturally predicted executable-wrong histories, with memory architecture unchanged.

## Directions removed from the active method pool

- **SampledRewriteUnsoundness / ReSequel:** analysis-only until a repair mechanism can be separated from established query-equivalence/test-suite machinery.
- **Semantic Repeatability Gap / SemBench:** analysis/evaluation object, not a method paper yet.
- **DownstreamImpact-GEM / GLEAM:** analysis/evaluation mismatch until a nontrivial matching objective is isolated beyond standard cost-sensitive entity resolution.
- **OpenSQL supervision fanout:** current obvious repair is verifier/test-suite module recombination.
- **Large-schema JoinBridge/ambiguity ideas:** remain artifact-blocked; no synthetic substitute is accepted as primary evidence.
- Generic active probing, generic multi-turn RL, generic memory, generic verifier/judge, generic query repair, generic schema linking, generic equivalence checking, generic candidate voting, and generic budget routing remain killed as standalone novelty.

## Local-registry evidence gap

The requested local paths `/mnt/liansp3/研究工作区/99_索引与规则/idea_source_registry.yaml` and `/mnt/liansp3/研究工作区/02_已审核/废案/REJECTED_IDEAS_20260825.md` were not present in the current runtime. These five ideas must be rechecked against that registry if it becomes accessible; changing a name will not be treated as sufficient to evade a collision.

## Canonical detailed method artifact

`research/text2sql-agent-fresh-2026-08-24/idea-stage/QUANT_METHOD_IDEAS_2026-08-27.md`

It contains five anchor limitation cards, exact-delta tables, input/intermediate/output definitions, primary/secondary metrics, strongest baselines, 20–50 example tests, full experiments, at most three causal ablations, kill criteria and 1–5 ratings.

## Older exploratory artifacts retained for provenance

- `ANCHOR_FAILURE_IDEAS_2026-08-26.md`
- `ANCHOR_FAILURE_IDEAS_DBVENUES_2026-08-26.md`
- `IDEA_POOL_EXPANSION_2026-08-26.md`
- `IDEA_REPORT.md`

These older files remain useful as provenance but do **not** override the method-only shortlist above.