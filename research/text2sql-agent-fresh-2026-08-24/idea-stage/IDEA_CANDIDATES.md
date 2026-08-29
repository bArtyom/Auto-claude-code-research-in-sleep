# Fresh Text2SQL × Agent Idea Candidates

**Updated:** 2026-08-29  
**Run:** independent fresh Text2SQL research branch  
**Current gate:** strong anchor → reproducible limitation → explicit algorithm delta → one primary quantitative metric → 20–50 example kill test.  
**Empirical pilots:** NOT RUN.  
**Formal independent reviewer:** UNAVAILABLE in this connector session; novelty remains provisional.

## Method-only shortlist — current priority

Only candidates with an explicit method change, fixed input/intermediate/output definition, strong baseline, primary metric and falsifiable low-cost test enter the active pool. A selected candidate is still killed if the deeper follow-up recheck reveals a direct prior-art collision.

| Rank | Method idea | Main anchor | Exact method object | Primary metric | First kill test | Status |
|---:|---|---|---|---|---|---|
| 1 | **Budgeted Multi-Candidate Distinguishing Database Selector (BMDD-Select)** | DPC, ACL 2026 | select a small test database that maximizes K-way execution partition information, then use one unchanged Python solver answer to update multiple SQL clusters | Top-1 EX @ fixed Python-solver-call budget | 40 released DPC BIRD candidate sets with ≥3 execution clusters | ACTIVE AFTER NOVELTY RECHECK |
| 2 | **Risk-Adaptive Execution Checkpointing** | VET, Findings ACL 2026 | learn per-step causal checkpoint criticality under a hard DB-call budget | EX@50% intermediate DB-call budget | 50 BIRD Moderate/Challenging traces | ACTIVE METHOD CANDIDATE |
| 3 | **Contrastive Value-Link Probe Selector** | DIVER, SIGMOD 2026 | choose DIVER probes by information gain over explicit value-link hypotheses | Value Linking F1@≤3 probes | 40 ambiguous BIRD value-link cases | ACTIVE METHOD CANDIDATE |
| 4 | **Error-Targeted SQL Synthesis Scheduler** | SQL-Factory, PVLDB 19(3) | allocate a fixed synthesis budget from the target student's residual error signatures | Held-out DB EX under equal synthetic budget G | 40 held-out questions + 2K-pair pilot | ACTIVE METHOD CANDIDATE |
| 5 | **Predicted-History Curriculum Training** | MTSQL-R1, ACL 2026 | train on naturally model-generated executable-wrong dialogue histories without changing memory architecture | Predicted-History EX | 40 natural error-carryover turns | ACTIVE, HIGHER INCREMENTAL RISK |

## Selected #1 recheck result

The previous rank-1 **Constraint-Conditioned MDD SQL Selector** is now **REJECTED AS A STANDALONE METHOD CLAIM**. The deeper recheck found:

- **ParSEval, PVLDB 2025** already generates SQL test databases under integrity constraints;
- older SQL test-generation/equivalence work already treats legal database states under constraints;
- **Data-aware candidate selection in NL2SQL translation via small separating instances** (arXiv:2605.12319, 2026) directly addresses unrealistic from-scratch separating instances by extracting small separating subsets from the real target database and performs a multi-candidate tournament.

Per the project rule, the old idea is not rescued by renaming it or by adding another agent. DPC is retained only because a different released-code limitation remains: after execution clustering it reduces the candidate set to one Champion–Challenger pair for verification.

The new rank-1 hypothesis is therefore narrower: under a fixed independent-verification budget, can one synthetic test database be chosen to **partition several live SQL hypotheses at once**, so one Python/Pandas answer eliminates multiple clusters and improves the accuracy–verification-cost frontier over DPC and matched-budget pairwise alternatives?

Detailed preregistered specification:

`research/text2sql-agent-fresh-2026-08-24/idea-stage/SELECTED_01_DPC_RECHECK_2026-08-29.md`

## Why the remaining four survive for now

2. **VET:** the paper explicitly identifies per-step DB waiting as a latency limitation. The idea optimizes checkpoint necessity inside the existing trace rather than introducing a generic query router.
3. **DIVER:** generic probing is crowded, but DIVER's tool-route ablation does not establish an explicit value-link hypothesis-discrimination objective under a fixed probe budget.
4. **SQL-Factory:** static structural curriculum already exists (e.g. SAC-SQL), so the surviving delta is closed-loop generation allocation from a particular student's measured residual errors under equal generation/training budgets.
5. **MTSQL-R1:** structured-history work such as Rose-SQL already exists, so the surviving delta is deployment-faithful training-state distribution over naturally predicted executable-wrong histories, with memory architecture unchanged.

## Directions removed from the active method pool

- **Constraint-Conditioned MDD SQL Selector:** killed by ParSEval + separating-instance follow-up collision; retained only in provenance.
- **SampledRewriteUnsoundness / ReSequel:** analysis-only until a repair mechanism can be separated from established query-equivalence/test-suite machinery.
- **Semantic Repeatability Gap / SemBench:** analysis/evaluation object, not a method paper yet.
- **DownstreamImpact-GEM / GLEAM:** analysis/evaluation mismatch until a nontrivial matching objective is isolated beyond standard cost-sensitive entity resolution.
- **OpenSQL supervision fanout:** current obvious repair is verifier/test-suite module recombination.
- **Large-schema JoinBridge/ambiguity ideas:** remain artifact-blocked; no synthetic substitute is accepted as primary evidence.
- Generic active probing, generic multi-turn RL, generic memory, generic verifier/judge, generic query repair, generic schema linking, generic equivalence checking, generic candidate voting, and generic budget routing remain killed as standalone novelty.

## Local-registry evidence gap

The requested local paths `/mnt/liansp3/研究工作区/99_索引与规则/idea_source_registry.yaml` and `/mnt/liansp3/研究工作区/02_已审核/废案/REJECTED_IDEAS_20260825.md` were not present in the current runtime. Active ideas must be rechecked against that registry if it becomes accessible; changing a name will not be treated as sufficient to evade a collision.

## Canonical detailed artifacts

- Original five-method report: `research/text2sql-agent-fresh-2026-08-24/idea-stage/QUANT_METHOD_IDEAS_2026-08-27.md`
- Selected #1 novelty recheck and replacement: `research/text2sql-agent-fresh-2026-08-24/idea-stage/SELECTED_01_DPC_RECHECK_2026-08-29.md`

## Stop boundary for selected #1

No empirical claim has been made. The next legitimate action is the locked 40-case BMDD-Select pilot specified in `SELECTED_01_DPC_RECHECK_2026-08-29.md`; if its simple pairwise controls match it, rank #1 is killed rather than extended with more modules.