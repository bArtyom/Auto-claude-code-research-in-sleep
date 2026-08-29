# Selected #1 — DPC novelty recheck and refined method

Date: 2026-08-29
Status: IDEA RECHECK COMPLETE; EMPIRICAL PILOT NOT STARTED.

## Decision

The previously selected **Constraint-Conditioned MDD SQL Selector** is rejected as a standalone novelty claim. ParSEval (PVLDB 2025) already generates SQL test databases under integrity constraints, and the 2026 preprint *Data-aware candidate selection in NL2SQL translation via small separating instances* directly addresses unrealistic synthetic separating worlds by extracting separating instances from the real target database. Per the kill-first rule, this direction is not rescued by renaming or by adding an agent.

The DPC anchor remains useful because the released method has a different, reproducible boundary: after execution clustering it verifies only one Champion–Challenger pair even when the candidate pool contains three or more execution-distinct hypotheses.

## Refined method

**Budgeted Multi-Candidate Distinguishing Database Selector (BMDD-Select)**

### Input
Natural-language question, target database/schema, K SQL candidates, maximum B independent Python-solver calls, and a fixed test-database context budget.

### Intermediate object
Execution clusters over candidates and a proposed small test database that induces a partition of all surviving candidate clusters by their execution result.

For a proposal D', score the induced partition using weighted information gain over cluster priors. Propose M small test databases, choose the proposal with maximum partition-information score, run the unchanged DPC Python/Pandas solver once, then update or eliminate every candidate cluster whose result is inconsistent with the Python answer.

### Output
Ranked candidate clusters and one final SQL, plus verification cost trace.

## Exact delta from DPC

- DPC verifies Champion vs Challenger only; BMDD keeps all execution-distinct clusters alive.
- DPC creates pair-focused MDDs; BMDD optimizes one test database for K-way candidate partition.
- DPC spends a solver call on one pair; BMDD uses one solver answer to update multiple clusters.
- The DPC solver remains unchanged in Gate A so any gain is attributable to multi-candidate test-database selection.

This is not a generic tournament: a tournament spends one comparison on one pair, while BMDD optimizes the database intervention itself so one semantic-solver call can distinguish several hypotheses.

## Primary metric

**Top-1 Execution Accuracy at B Python-solver calls**, with pilot B=2.

## 40-case kill test

Use the official DPC candidate snapshot `results/candidates/Qwen2.5-Coder-7B-Instruct_BIRD_Mini_Dev.json`; do not generate new candidates.

Eligibility is fixed before method evaluation:
1. Oracle@K = 1 (at least one released candidate is execution-correct);
2. at least 3 distinct execution clusters on the original DB;
3. every selector receives the identical candidate list.

Deterministically sample 40 eligible cases and lock the IDs before evaluation.

Matched baselines:
1. released DPC;
2. self-consistency / majority execution cluster;
3. random-pair DPC with B=2;
4. greedy-pair DPC with B=2 (always duel the two largest surviving clusters);
5. exhaustive pairwise DPC as a high-cost upper reference.

Pilot success requires all of:
- BMDD is at least 2/40 correct above original DPC;
- BMDD is at least 2/40 above the stronger matched-budget pairwise baseline;
- median surviving clusters drop by at least 50% after the first selected K-way test database;
- verification tokens are at most 1.5x original DPC.

Kill immediately if any of these hold:
- gain over DPC is at most 1/40;
- greedy second-duel DPC is within one case of BMDD;
- partition-information proposal selection is no better than random proposal selection;
- exhaustive pairwise DPC does not materially outperform original DPC on the eligible subset, implying no multi-cluster headroom;
- most proposed K-way databases induce only two execution groups;
- a newer paper already performs equivalent no-gold K-way distinguishing-instance selection with an independent semantic solver under a fixed verification budget.

## Full experiment only if pilot passes

Datasets: BIRD Mini-Dev and Spider released DPC candidate pools. Use at least two released generators (Qwen2.5-Coder-7B plus DeepSeek-Chat or OmniSQL-7B). Report Top-1 EX versus Python-solver calls and verification tokens, candidate-cluster elimination rate per solver call, candidate MRR, token cost per successful selection, and latency. Use paired bootstrap 95% CIs and McNemar tests.

Maximum three ablations:
1. information-gain proposal selection -> random valid proposal;
2. K-way update -> pairwise update using the same selected test database;
3. frequency-weighted prior -> uniform prior.

## Prior-art guardrail

Do not claim first SQL test-database generation, first constraint-aware test generation, first separating instance, first candidate tournament, or first multi-candidate Text-to-SQL selector. The only claim worth testing is whether K-way discriminative database selection improves the selection-accuracy / independent-verification-cost frontier over DPC and matched-budget pairwise alternatives.

## Stop boundary

Idea-stage refinement ends here. The next legitimate action is empirical: materialize the locked 40-case manifest, reproduce DPC, implement the minimal K-way Tester/pipeline delta, and run the preregistered pilot.