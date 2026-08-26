# Anchor-Paper Failure-Boundary Idea Expansion — Text2SQL × Database Agents

**Date:** 2026-08-26  
**Branch:** `research/text2sql-agent-fresh-2026-08-24`  
**Method:** anchor paper → core hypothesis → failure boundary → single-variable falsification → only then method idea  
**Empirical pilots:** NOT RUN  
**Formal cross-model review:** UNAVAILABLE in this connector session; novelty is provisional.

## 0. Rules for this round

This expansion intentionally rejects method-first ideation. Every surviving idea must satisfy all of the following:

1. start from an observed or explicitly claimed phenomenon in a strong recent paper/benchmark;
2. isolate one assumption or failure dimension;
3. have a 20–50 example cheap reproduction/falsification experiment before full implementation;
4. use an existing public benchmark/repository when possible rather than inventing a synthetic-only task;
5. state a kill condition before proposing a large architecture;
6. treat old database/program-analysis theory as prior art, not as novelty merely because an LLM is added.

The selected anchors are deliberately reproducible and methodologically different:

- **BIRD-INTERACT** — ICLR 2026 Oral; public benchmark + ADK implementation.
- **SQL-Trail** — ACL 2026 Main; multi-turn RL with execution feedback and adaptive turn budget.
- **DPC** — ACL 2026 Main; Minimal Distinguishing Database + Python/Pandas cross-paradigm candidate selection; public code.
- **VET** — Findings ACL 2026; step-wise executable Python semantics against the real DB.
- **Spider 2.0 / Spider2-DBT** — ICLR 2025 Oral; public enterprise SQL/code-agent benchmark.
- **MTSQL-R1** — ACL 2026 Main; public code/models for long-horizon multi-turn propose→execute→verify→refine.

---

# 1. Anchor evidence cards

## A1 — BIRD-INTERACT (ICLR 2026 Oral)

**Core hypothesis.** Real database assistance requires dynamic interaction rather than static dialogue context. The benchmark couples databases with hierarchical knowledge, metadata, an executable environment and a function-driven user simulator; a-Interact lets agents decide when to ask/explore.

**Evidence/reproducibility.** Public GitHub/HuggingFace data, Mini/Lite/Full variants, ADK-based modular environment. Dataset records expose ambiguity labels, preprocess/cleanup SQL, follow-ups and test cases.

**Explicit boundary from paper.** The authors state two future-work items: a more human-aligned local user simulator and a free-mode evaluation without the current strict budget stress. Therefore simulator realism and budget regime are not closed questions.

**Do not propose.** Another clarification policy, generic interaction agent, generic memory, or adaptive-turn method.

## A2 — SQL-Trail (ACL 2026 Main)

**Core hypothesis.** Multi-turn database interaction plus execution feedback and adaptive turn allocation closes the gap left by single-pass generation.

**Evidence/reproducibility.** ACL paper provides matched benchmark evaluation; the method has explicit reward design around correctness and efficient exploration.

**Explicit boundary from limitations.** It assumes an interactive execution environment; database access can be unavailable/expensive/private, and multi-turn execution increases latency/cost. More subtly, the method treats execution feedback as useful supervision, while execution correctness on one database is known to admit semantic false positives.

**Do not propose.** Generic more-turns, execution feedback, query repair, turn router, or exploration reward.

## A3 — DPC (ACL 2026 Main)

**Core hypothesis.** Candidate selection improves when competing SQLs are forced to diverge on a synthesized Minimal Distinguishing Database (MDD), then checked through an independent Python/Pandas solution.

**Evidence/reproducibility.** Public code. DPC reports up to +2.2 absolute accuracy over strong training-free selectors on BIRD/Spider.

**Critical assumption.** The MDD must be discriminative, but the paper's construction criterion is primarily contextual feasibility + candidate divergence. One published running example creates a "Ghost Transaction" that lacks a corresponding `yearmonth` row to separate candidates. Whether such a world is valid under the real application's integrity/business constraints is not the same question as whether both SQLs execute on it.

**Prior-art guardrail.** Constraint-based test database generation is old, and ParSEval (PVLDB 2025) already generates plan-aware distinguishing databases. Therefore `add a constraint solver` is not novelty. The research object must be **whether Text2SQL verifier conclusions change when counterexamples are restricted to realizable enterprise worlds**.

## A4 — VET (Findings ACL 2026)

**Core hypothesis.** Step-wise executable semantics make reasoning observable and reduce hallucination; each Python step is executed against the real DB.

**Evidence/reproducibility.** Reports 70.93% EX on BIRD and 37.04% on Spider2-Lite in matched training-free evaluation.

**Explicit limitations.** Step-wise execution adds latency, and direct database/Python access may be disallowed for sensitive databases. Also, cross-paradigm semantics are not automatically identical: Pandas documentation explicitly notes null-join behavior differs from SQL, and 2026 Text-to-Python work already documents SQL/Pandas mismatch.

**Do not propose.** Generic Python verifier or `SQL + Python consistency`; this is already crowded.

## A5 — Spider 2.0 (ICLR 2025 Oral)

**Core hypothesis.** Enterprise Text2SQL is a workflow/code-agent problem involving huge schemas, dialect documentation, external artifacts, codebases, multiple queries and long contexts.

**Evidence/reproducibility.** Public Spider2-Snow/Lite/DBT repositories and benchmark. The repository also retains data/evaluation update history; official notes document ambiguity fixes, evaluator fixes and environment/credential changes.

**Boundary.** Spider2 gives realistic artifacts, but usually evaluates one static benchmark snapshot. Real enterprise documentation, dbt code and data change over time, and benchmark history itself demonstrates that environment artifacts evolve.

**Do not propose.** Another schema linker, long-context compressor, generic repository-aware agent or dialect prompt.

## A6 — MTSQL-R1 (ACL 2026 Main)

**Core hypothesis.** Long-horizon multi-turn reasoning with DB execution and persistent dialogue memory improves coherence and correction over short-horizon parsing.

**Evidence/reproducibility.** Public GitHub; code, training framework and models. Repo analysis reports stronger gains on long/complex turns and notes that smaller models struggle with long-horizon function calling.

**Boundary.** Persistent history is helpful when history is trustworthy. A long-horizon system may instead amplify an early plausible-but-wrong SQL through later turns. This failure can be tested by controlled intervention without inventing a new dataset.

---

# 2. New idea bank — derived from failure boundaries, not method names

## Cluster A — When execution feedback is a bad teacher

### 1. **DenotationRewardTrap-SQL**

**Failure boundary.** SQL-Trail/ReEx/Arctic-style RL commonly rewards execution success. A semantically wrong query can match the gold denotation on one database snapshot by coincidence; test-suite evaluation has documented this issue for years.

**Hypothesis.** Multi-turn execution-RL policies can learn snapshot-specific shortcuts because false-positive denotations are rewarded as success; the problem is larger for interactive agents because they can probe and exploit current data quirks.

**Cheap test.** Take 20–50 examples judged successful by single-snapshot execution from public RL Text2SQL outputs. Re-score with Spider's official distilled test suites / ParSEval-generated distinguishing instances. Compare false-positive reward rate before and after RL and inspect whether interactive policies are more brittle than direct models.

**Kill if.** False-positive reward rate is negligible (<~3–5%) or does not increase/affect learned behavior.

**Paper shape if positive.** `Agentic Denotation Overfitting in Execution-Rewarded Text-to-SQL`, followed by a small counterfactual/test-suite reward rather than a large new agent architecture.

**Prior-art warning.** Test-suite accuracy is EMNLP 2020 prior art; Graph-Reward-SQL already criticizes execution rewards. Novelty must be the *agentic RL failure mechanism and policy change*, not the metric itself.

### 2. **Feedback-Information Boundary (FIB-SQL)**

**Failure boundary.** Execution feedback is treated as a generic benefit, but different outcomes convey radically different information: syntax error, missing column, timeout, empty result, plausible non-empty result, and coincidentally correct result.

**Hypothesis.** Required interaction depth is determined less by nominal question difficulty than by the information content of observed feedback.

**Cheap test.** On 30–50 SQL-Trail/ReEx trajectories, label feedback classes and measure probability of successful next-step correction. Pair tasks of similar SQL complexity but different feedback types. Estimate entropy reduction / candidate elimination after each feedback event.

**Kill if.** Feedback class explains little after controlling for task difficulty, or a simple error-category baseline already captures everything.

**Potential delta.** Replace difficulty-based turn budget with observed *feedback informativeness* only if the boundary is real.

### 3. **OperationalFeedback-SQL**

**Failure boundary.** Production execution engines return timeout, bytes-scanned, quota, permission and resource errors. These do not necessarily imply that SQL semantics are wrong. An agent trained to always "repair after error" can silently change the requested meaning to satisfy operations constraints.

**Hypothesis.** Operational feedback causes measurable semantic drift in execution-feedback agents.

**Cheap test.** Take 20–30 correct Spider2/BigQuery/Snowflake queries; induce realistic timeout/scan-budget conditions while preserving query semantics. Feed only the operational error to an execution-feedback agent and check whether the revised SQL remains semantically equivalent.

**Kill if.** Agents overwhelmingly preserve semantics or existing type-specific repair baselines already solve it.

**Prior-art warning.** SIRIUS-SQL already has type-specific timeout repair, so a paper needs evidence of *semantic damage under operational feedback*, not another error classifier.

### 4. **FutureSnapshot-SQL**

**Failure boundary.** Current benchmarks reward correctness on a static data state. A wrong SQL can accidentally be correct now yet fail after ordinary future inserts while schema/business semantics remain unchanged.

**Hypothesis.** Static-snapshot execution materially overestimates deployable semantic correctness of modern agents, especially execution-guided/RL systems.

**Cheap test.** Find 20–50 tasks over real timestamped/public datasets where natural historical/future slices can be reconstructed. Generate SQL on snapshot t, then evaluate the same SQL against t+1/t+2 while recomputing gold semantics. No schema mutation.

**Kill if.** Performance is stable after controlling for genuinely time-dependent questions.

**Why this is distinct.** Unlike random test-suite generation, the failure is validated on *natural data evolution* rather than synthetic database worlds.

---

## Cluster B — When the verifier's counterexample world is impossible

### 5. **WorldValid-MDD / RealizableCounterexample-SQL**

**Failure boundary.** DPC only needs a small synthetic database where candidates diverge; a discriminating database can violate real integrity/domain/business invariants and therefore be irrelevant to the actual question.

**Hypothesis.** A non-trivial fraction of DPC's winning MDDs distinguish candidates using impossible or implausible states, and selector accuracy changes when only realizable worlds are allowed.

**Cheap test.** Reproduce 30–50 DPC candidate duels. For each generated MDD, check DDL PK/FK/NOT NULL/CHECK constraints first, then optional mined FDs/value-domain invariants. Regenerate MDDs with increasing constraint sets and measure winner flip rate and gold selection accuracy.

**Kill if.** Almost all DPC MDDs already satisfy realistic constraints, or constraint enforcement does not alter decisions.

**Strongest novelty statement.** Not "constraint-aware test generation"; instead: **Text2SQL verification can be unsound because it reasons over counterexample worlds that cannot occur in the target warehouse.**

### 6. **MDD-Stability-SQL**

**Failure boundary.** DPC needs *an* MDD, but many different distinguishing databases may exist. If different valid MDDs cause the Python solver/selector to choose different candidates, selection depends on counterexample choice rather than user intent.

**Hypothesis.** Candidate decisions exhibit non-trivial variance across independently generated valid MDDs for the same duel.

**Cheap test.** For 30–50 DPC duels, generate 5–10 MDDs with different seeds/prompts while holding schema and candidate pair fixed. Record selection entropy, Python-solver disagreement and gold accuracy.

**Kill if.** >95% of decisions are invariant across valid MDDs.

**If positive.** The method contribution can be small: choose counterexamples by stability/worst-case criterion, not just first divergence.

### 7. **Constraint Acquisition Ladder for Verifiers**

**Failure boundary.** DDL constraints are incomplete; enterprise invariants also live in dbt tests, documentation, data profiles and code.

**Hypothesis.** The marginal value of verifier constraints has a sharp hierarchy: DDL-only may be insufficient, while a small set of repository-derived invariants removes most invalid counterexamples.

**Cheap test.** On the same DPC subset, incrementally add: DDL → dbt tests → discovered keys/FDs → documentation rules. Measure invalid-MDD rate and selector accuracy after each layer.

**Kill if.** DDL alone captures almost all benefit or extra metadata does not change verification.

**Paper shape.** Analysis/benchmark first; only build automatic constraint retrieval if the ladder shows a clear missing layer.

### 8. **Cross-Paradigm Oracle Audit — ATTACK ONLY, NOT PRIMARY IDEA**

DPC/VET use Python/Pandas as a high-confidence independent reasoning path, but Pandas itself documents SQL-semantic differences (e.g. NULL keys match in `merge`), and recent 2026 Text-to-Python work explicitly studies semantic mismatch. Use this as a robustness attack/ablation for DPC, not as a standalone novelty claim.

---

## Cluster C — When long-horizon interaction amplifies rather than repairs errors

### 9. **ErrorCarryover-SQL**

**Failure boundary.** MTSQL-R1's persistent memory improves coherence under normal histories. What happens after one earlier SQL is executable but semantically wrong?

**Hypothesis.** Long-horizon agents exhibit *error amplification*: one early wrong state has a larger downstream effect than on short-horizon baselines, and later execution feedback often fails to recover because the mistaken semantic premise remains coherent.

**Cheap test.** Select 30–50 CoSQL/SParC dialogues. At one early turn, inject a controlled plausible wrong SQL/memory state while keeping subsequent user utterances unchanged. Compare downstream accuracy for MTSQL-R1, a history-only baseline, and a stateless-per-turn baseline.

**Metrics.** downstream error area-under-curve, recovery turns, probability of repeating the same wrong schema/constraint, and damage after the injected turn.

**Kill if.** MTSQL-R1 recovers within one turn or is no more vulnerable than short-horizon baselines.

**Why stronger than generic memory poisoning.** The intervention is *inside a validated multi-turn agent* and measures causal propagation of one specific history error, not a new memory architecture.

### 10. **Correction-vs-Goal-Shift Boundary**

**Failure boundary.** Multi-turn datasets often encode edits to a stable intent, but real users sometimes retract or replace the goal itself.

**Hypothesis.** Long-horizon agents over-preserve prior intent when a new turn is a goal replacement rather than an incremental SQL edit.

**Cheap test.** First mine existing SParC/CoSQL/BIRD-Interact cases with explicit removal/replacement operations; do not synthesize yet. Categorize incremental edit vs true goal replacement and evaluate memory-heavy models separately.

**Kill if.** Existing datasets contain too few true goal shifts to support the claim; do not manufacture a synthetic benchmark as the primary evidence.

### 11. **Small-Model Tool Bottleneck Audit**

**Failure boundary.** MTSQL-R1 repo reports small models struggle with long-horizon function calling. The failure may be tool syntax rather than SQL semantics.

**Hypothesis.** A substantial fraction of the small-model gap disappears when tool actions are grammar-constrained/oracle-formatted while the same semantic reasoning burden remains.

**Cheap test.** On 20–50 difficult multi-turn tasks, compare free-form tool calls vs constrained action serialization for 1.7B/4B checkpoints.

**Kill if.** Semantic accuracy remains equally low after tool-format errors are removed.

**Value.** Prevents misattributing a tool-interface bottleneck to reasoning ability.

---

## Cluster D — When the interactive benchmark itself changes the policy being measured

### 12. **BudgetDistortion-BIRD**

**Failure boundary.** BIRD-INTERACT explicitly describes a-Interact as a strict budget stress mode and lists free-mode evaluation as future work.

**Hypothesis.** Agent ranking and qualitative strategy are not invariant to the budget regime; a policy optimized for stress mode can look superior while being inferior when user/DB interaction is cheap.

**Cheap test.** Use 30–50 Mini/Lite tasks and 3–5 existing agents. Run identical agents under tight, medium and effectively-free interaction budgets. Compare rank correlation, success, user questions, DB probes and cost.

**Kill if.** Rankings and strategies remain stable (e.g. Spearman >0.95, small effect sizes).

**Paper shape if positive.** Benchmark/evaluation paper first. A budget-robust agent is optional follow-up, not required for the initial claim.

### 13. **SimulatorTransfer-BIRD**

**Failure boundary.** BIRD uses a function-driven simulator specifically to reduce leakage/unfair behavior. Interaction policies may nevertheless specialize to that channel.

**Hypothesis.** Success depends materially on the simulator realization even when underlying symbolic user facts are unchanged.

**Cheap test.** Keep the same BIRD symbolic user actions but render them via: canonical template, paraphrased natural-language response, terse response, and one alternative LLM verbalizer. Measure strategy/success transfer.

**Kill if.** Agent policies are invariant across realizations.

**Claim restriction.** This only establishes *simulator transfer*, not human transfer. Do not claim human realism without real users.

### 14. **Interaction-State Damage Score**

**Failure boundary.** BIRD-INTERACT includes CRUD and uses preprocess/cleanup SQL. A task may eventually pass while the agent caused unnecessary intermediate state mutations.

**Hypothesis.** End-task success hides large differences in transient database damage/risk across agents.

**Cheap test.** On write-enabled BIRD tasks, replay agent trajectories and compute state deltas after each mutation relative to the minimal gold transition.

**Kill if.** Agents almost never issue extra mutations or the score collapses to ordinary task success.

**Prior-art warning.** Generic agentic transactions/Cordon already exist; novelty can only be a Text2SQL evaluation object grounded in BIRD's actual write trajectories.

---

## Cluster E — When enterprise artifacts are stale or structurally deep

### 15. **DocDrift-SQL**

**Failure boundary.** Spider2 explicitly requires metadata/documentation/code artifacts, but evaluates a curated static snapshot. Real enterprise documentation becomes stale.

**Hypothesis.** Modern enterprise Text2SQL agents are substantially more vulnerable to *stale but plausible documentation* than to missing documentation, because stale docs create confident semantic misgrounding.

**Cheap test.** Mine Spider2 Git history/data-update logs and public dbt repositories for 20–50 cases where schema/docs changed over time. Pair current DB state with genuinely older documentation when possible; compare current-doc, stale-doc, no-doc and execution-only conditions.

**Kill if.** Real historical changes are too sparse to construct a credible set, or stale docs do not hurt more than simply removing docs.

**Why this respects the evidence rule.** Prefer real artifact history; only use synthetic staleness as secondary stress testing.

### 16. **ArtifactConflict-SQL**

**Failure boundary.** Enterprise tasks can expose schema, docs, dbt models and runtime data simultaneously; these sources can disagree.

**Hypothesis.** Agents have unstable implicit source-precedence rules, and correctness depends on which artifact is wrong/stale.

**Cheap test.** Start only from real DocDrift cases where two artifacts disagree. Do not generate artificial conflicts until a real phenomenon is verified. Track which source agents follow.

**Kill if.** Too few real conflicts exist or a deterministic `runtime schema > docs` rule solves nearly all cases.

### 17. **RepoLineageDepth-SQL**

**Failure boundary.** Spider2-DBT requires repository/code navigation, but difficulty is typically reported at task level rather than structural dependency depth.

**Hypothesis.** Failure exhibits a sharp cliff as the required transformation lineage/macro-indirection depth increases, even after controlling for prompt length and SQL length.

**Cheap test.** On all 68 Spider2-DBT tasks, compute dbt DAG depth, number of referenced models/macros, and expansion depth. Regress existing agent success against these structural features.

**Kill if.** No stable relationship survives simple controls.

**If positive.** Only then design a depth-aware decomposition method; the first result can itself be a boundary/analysis paper.

### 18. **FormatAnchoring Failure in ReFoRCE**

**Failure boundary.** ReFoRCE predicts expected answer format early and reinforces it during refinement.

**Hypothesis.** Incorrect early format predictions causally lock the agent into the wrong semantic interpretation.

**Cheap test.** On 30–50 Spider2 tasks, run ReFoRCE with predicted format, no format, oracle format, and minimally perturbed wrong format. Measure downstream SQL changes and recovery.

**Kill if.** Wrong format has little causal effect or self-refinement reliably overrides it.

**Value.** A clean failure-boundary experiment; likely not enough novelty by itself unless effect is large and general.

---

# 3. Twenty-five compact secondary ideas

These are lower-priority until an anchor failure is reproduced.

| # | Idea | Anchor failure variable | Cheap first test |
|---:|---|---|---|
| 19 | **EmptyResultTrust-SQL** | correct-empty vs wrong-empty feedback ambiguity | isolate genuine empty gold tasks and replay repair agents |
| 20 | **TimeoutSemanticDrift** | timeout repair changes meaning | inject timeouts into known-correct complex queries |
| 21 | **InteractionShortcut-SQL** | agent learns current-data literals/quirks via probes | future-snapshot replay |
| 22 | **CounterexampleWorldDistance** | synthetic verifier DB far from target data manifold | compare MDD profile statistics vs real DB |
| 23 | **MDD Business-Rule Coverage** | DDL constraints miss business invariants | add dbt tests/docs one source at a time |
| 24 | **Verifier Abstain-on-Unrealizable** | no valid discriminating world found | measure candidate pairs indistinguishable under known constraints |
| 25 | **History Error Half-Life** | persistence duration of injected prior error | error intervention at different dialogue positions |
| 26 | **Recovery Trigger Benchmark** | what observation finally breaks wrong dialogue state | log first corrective evidence after injection |
| 27 | **Tool-Syntax Normalization Gap** | tool protocol vs reasoning competence | constrained vs free action format |
| 28 | **Budget Rank Instability Score** | evaluation regime dependence | tight/free BIRD interaction matrix |
| 29 | **Clarification Channel Sensitivity** | same user fact, different surface response | simulator render variants |
| 30 | **Write Amplification Factor** | extra state mutations per successful task | BIRD CRUD trajectory replay |
| 31 | **Cleanup Dependence Audit** | success relies on evaluator cleanup/reset assumptions | run selected tasks without cleanup between attempts in sandbox |
| 32 | **StaleDocs-vs-NoDocs Gap** | misinformation worse than missing information | historical Spider2/dbt docs |
| 33 | **Documentation Authority Calibration** | model trust in docs vs runtime evidence | real artifact conflicts only |
| 34 | **Repo Macro Depth** | macro expansion rather than schema size | Spider2-DBT structural analysis |
| 35 | **Repository Dead-Code Distractors** | irrelevant but plausible dbt models | first measure natural unused-model density |
| 36 | **Early Output-Shape Lock-In** | predicted answer shape biases semantic plan | ReFoRCE format ablation |
| 37 | **Stepwise Verification Redundancy** | VET executes many non-diagnostic steps | leave-one-step-out replay on logged traces |
| 38 | **Critical Checkpoint Coverage** | only certain semantic decisions need execution | correlate step execution with error discovery |
| 39 | **Private-DB Verification Degradation** | execution tracing restricted to views/samples | VET on permission-limited replicas if available |
| 40 | **Cross-Dialect Result Stability** | same intent, dialect semantics differ | only if paired cross-dialect tasks can be obtained without synthetic core |
| 41 | **Benchmark Revision Sensitivity** | leaderboard ranking changes with evaluator/data versions | replay stored predictions on Spider2 historical evaluator revisions |
| 42 | **Environment Reproducibility Debt** | cloud/credential/runtime changes affect score | quantify non-model failure rate from public issue/history logs |
| 43 | **Gold-Table Dependence Curve** | schema retrieval vs SQL reasoning confounding | full schema → top-k → oracle table ablation on Spider2 |

---

# 4. Current anchor-driven shortlist

The ranking below prioritizes: real public evidence path, one isolated variable, low-cost kill test, and distance from already-crowded mechanisms.

| Rank | Idea | Why it is interesting | Novelty pressure | First kill cost |
|---:|---|---|---|---|
| 1 | **WorldValid-MDD** | directly attacks an assumption of a new ACL-2026 verifier using real integrity constraints | medium: old DB testing theory is strong | low |
| 2 | **ErrorCarryover-SQL** | tests whether long-horizon agents amplify one earlier semantic error | relatively low | low |
| 3 | **BudgetDistortion-BIRD** | exactly targets an ICLR-2026 acknowledged benchmark boundary | low-medium | low |
| 4 | **FutureSnapshot-SQL** | replaces synthetic equivalence stress with natural data evolution | medium | medium |
| 5 | **DenotationRewardTrap-SQL** | execution-RL may optimize wrong semantics through false-positive rewards | high: test-suite/reward literature exists | low-medium |
| 6 | **DocDrift-SQL** | enterprise agents rely on docs; stale docs are realistic and under-tested | medium | medium |
| 7 | **MDD-Stability-SQL** | verifier output may depend on arbitrary counterexample choice | medium-low | low |
| 8 | **RepoLineageDepth-SQL** | isolates a structural enterprise failure dimension before inventing a planner | low | very low |
| 9 | **Feedback-Information Boundary** | asks what execution feedback actually teaches | medium-high | low |
| 10 | **SimulatorTransfer-BIRD** | tests policy dependence on simulator realization without claiming human transfer | medium | medium |

---

# 5. Recommended next sequence — still idea stage

Do **not** implement new methods yet. The next work should be five reproduction probes, each on 20–50 existing examples:

1. **DPC:** regenerate MDDs and audit realizability + selection stability.
2. **MTSQL-R1:** inject one plausible prior-turn semantic error and measure downstream recovery/amplification.
3. **BIRD-Interact:** rerun the same tasks under tight vs free-ish interaction budgets.
4. **Execution-RL:** compare single-snapshot success against official test-suite/perturbed-world correctness.
5. **Spider2:** mine repository/data history for naturally stale documentation cases before designing any DocDrift method.

The decision rule is deliberately harsh:

- if the anchor method survives the probe, kill the derived idea;
- if a simple deterministic baseline explains the effect, kill the method contribution and possibly keep only an analysis result;
- if the phenomenon only appears under fabricated synthetic cases, do not use it as the primary paper claim;
- only after a robust failure boundary is observed should a repair mechanism be designed.

This is the main methodological change from the earlier idea pool: **the next idea is not selected because the proposed mechanism sounds new; it is selected because a strong published system fails in a reproducible, narrow, scientifically interpretable way.**

## Sources / anchors

- BIRD-INTERACT, ICLR 2026 Oral: https://openreview.net/forum?id=nHrYBGujps ; https://github.com/bird-bench/BIRD-Interact
- SQL-Trail, ACL 2026 Main: https://aclanthology.org/2026.acl-long.1677/
- DPC, ACL 2026 Main: https://aclanthology.org/2026.acl-long.313/ ; https://github.com/HKUSTDial/DPC
- VET, Findings ACL 2026: https://aclanthology.org/2026.findings-acl.1544/
- MTSQL-R1, ACL 2026 Main: https://aclanthology.org/2026.acl-long.1563/ ; https://github.com/taichengguo/MTSQL-R1
- Spider 2.0, ICLR 2025 Oral: https://proceedings.iclr.cc/paper_files/paper/2025/hash/46c10f6c8ea5aa6f267bcdabcb123f97-Abstract-Conference.html ; https://github.com/xlang-ai/Spider2
- Test-suite semantic evaluation: https://aclanthology.org/2020.emnlp-main.29/
- ParSEval, PVLDB 2025: https://www.vldb.org/pvldb/vol18/p4750-miao.pdf
