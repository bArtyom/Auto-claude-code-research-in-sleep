# Quantitative Method Ideas — Anchor Paper → Limitation → Algorithm → Kill Test

**Date:** 2026-08-27  
**Branch:** `research/text2sql-agent-fresh-2026-08-24`  
**Stage:** idea discovery only; no empirical experiment has been run.  
**Selection rule:** method-type ideas only. Analysis-only failures, benchmark-only proposals, generic Agent/RAG/judge additions, and directions that collapse to established query-equivalence/test-suite machinery are excluded.

## Evidence limitation

The requested local files were not available in the current runtime:

- `/mnt/liansp3/研究工作区/99_索引与规则/idea_source_registry.yaml`
- `/mnt/liansp3/研究工作区/02_已审核/废案/REJECTED_IDEAS_20260825.md`

Therefore this pass cannot claim registry-level deduplication against those two files. Deduplication instead used the current fresh branch's rejected/saturated directions plus a fresh follow-up literature search. Any later run with access to the local registry must re-check the five survivors before implementation.

---

# 0. Five anchor limitation cards

## A1. DPC — ACL 2026 Main

**Paper.** Boyan Li, Ou Ocean Kun Hei, Yue Yu, Yuyu Luo. *DPC: Training-Free Text-to-SQL Candidate Selection via Dual-Paradigm Consistency*. ACL 2026 Main.  
Paper: https://aclanthology.org/2026.acl-long.313/  
Code: https://github.com/HKUSTDial/DPC

**Task.** Given a natural-language question, schema, and K SQL candidates, select the single correct candidate without verifier training.

**Core method.** DPC clusters candidate executions, selects a champion/challenger pair, synthesizes a Minimal Distinguishing Database (MDD), solves the question independently in Python/Pandas on that MDD, and selects the SQL whose result best matches the Python result. The strongest matched training-free baseline in the paper is Self-Consistency; on BIRD with Qwen2.5-Coder-7B, DPC reports 47.6 EX vs 46.4 for Self-Consistency, with ~3.8K tokens and ~4.2 s average selection latency.

**Reproducible boundary.** The paper explicitly states that under complex implicit constraints, synthesized environments may structurally diverge from real distributions, and improving semantic fidelity is future work. The MDD formalization requires contextual feasibility and discriminative validity, but not satisfaction of target PK/FK/UNIQUE/CHECK or trusted business invariants.

**Follow-up/prior-art check.** Constraint-based database generation, SQL equivalence under constraints, and distinguishing-test generation already exist; therefore `add a constraint solver` is not novelty. I did not find a direct 2026 follow-up that conditions DPC's inference-time candidate selection on target-realizable MDDs. The defensible delta is the **selection policy under realizability constraints**, including abstaining from an unrealizable duel.

---

## A2. VET — Findings ACL 2026

**Paper.** Dongyu Wang, Jingyu Li, Lan Zhang, Ganggang Yu, Liang Huang. *VET: Verifiable Execution Tracing for Reliable Text-to-SQL Generation*. Findings ACL 2026.  
Paper: https://aclanthology.org/2026.findings-acl.1544/  
Public evaluation data/protocol: BIRD, Spider/Spider-DK/Spider-SYN, Spider 2.0-lite. No official VET repository was found in this search, but the paper specifies temperature, step/repair limits, prompts/protocol, and public benchmarks.

**Task.** Training-free Text-to-SQL generation with step-wise executable reasoning.

**Core method.** VET constrains reasoning to a candidate schema and executes every generated Python reasoning step against the real database; execution feedback triggers repair, followed by constrained SQL reconstruction. With DeepSeek-Chat on BIRD, the strongest matched baseline RSL-SQL is 63.56 EX and VET reaches 68.38; with GPT-4o, 67.21 vs 70.93.

**Reproducible boundary.** The paper explicitly lists step-wise database waiting as a latency bottleneck. It permits up to five reasoning steps and three repairs per step. On challenging BIRD examples, reported average time/token use is 56.5 s / 17K tokens. VET does not learn which intermediate steps are actually worth executing.

**Follow-up/prior-art check.** SDE-SQL/PV-SQL already study database probing, and BAP-SQL (Aug 2026) studies query-level budget-aware observation planning. A generic `execute fewer probes` idea is therefore crowded. The remaining delta is **step-level checkpoint selection inside a fixed VET trace**, learned from causal leave-one-checkpoint-out evidence under a hard DB-call budget.

---

## A3. DIVER — SIGMOD 2026

**Paper.** Yafeng Nan, Haifeng Sun, Zirui Zhuang, Qi Qi, Guojun Chu, Jianxin Liao, Dan Pei, Jingyu Wang. *DIVER: A Robust Text-to-SQL System with Dynamic Interactive Value Linking and Evidence Reasoning*. SIGMOD 2026.  
Paper: https://arxiv.org/abs/2602.12064  
Code: https://github.com/thatmee/DIVER

**Task.** Expert-free evidence construction for Text-to-SQL, especially schema/value linking over large and dynamic database contents.

**Core method.** DIVER decomposes the NL question, then a Lookup Assistant operates an eight-tool database toolbox inside a Chain-of-Thoughts-and-Facts workspace, iteratively selecting probes and recording facts before evidence generation. On BIRD linking evaluation, DIVER reports overall Value Linking F1 79.53 vs CodeS 79.21; on challenging cases it reports 72.87 vs 67.58. Toolbox ablations show `uniq_value` and `sim_columns` are important, while removing the learned/prompted tool route does not lower EX and even raises VES in the reported ablation.

**Reproducible boundary.** DIVER's next tool is selected through open-ended LLM reasoning rather than an explicit objective over competing value-link hypotheses. The tool-routing ablation suggests that the routing mechanism itself is not tightly tied to final utility, and repeated/redundant calls can occur.

**Follow-up/prior-art check.** SDE-SQL, PV-SQL, and BAP-SQL make generic active probing/budget control crowded. Therefore the idea below is not another observation planner: it defines an explicit **hypothesis-discrimination objective for value linking**, with Value Linking F1 under a hard probe budget as the primary metric.

---

## A4. SQL-Factory — PVLDB 19(3), 2025 (VLDB 2026 cycle)

**Paper.** Jiahui Li, Tongwang Wu, Yuren Mao, Yunjun Gao, Yajie Feng, Huaizhong Liu. *SQL-Factory: A Multi-Agent Framework for High-Quality and Large-Scale SQL Generation*. PVLDB 19(3):292–305, 2025.  
Paper: https://www.vldb.org/pvldb/vol19/p292-gao.pdf  
Code/artifacts: https://github.com/LJHzju/SQL-Factory

**Task.** Generate large, structurally diverse, executable SQL corpora cheaply for downstream intelligent-database tasks such as Text-to-SQL training.

**Core method.** A Generation Team explores structures, an Expansion Team scales promising patterns, and a Management Team schedules generation based on real-time query quality, schema coverage, and under-covered/structurally rich tables. The paper generates >300K queries for <$200 and shows downstream gains; e.g. Qwen2.5-Coder-1.5B improves from 69.2 to 76.5 EX on Spider and from 28.6 to 40.9 on BIRD after SQL-Factory data training. For generation, SQLSmith/LearnedSQLGen/OmniSQL are comparison baselines; for downstream Text-to-SQL, official benchmark training data is a strong matched data baseline.

**Reproducible boundary.** Scheduling optimizes schema coverage/diversity/quality, not the current student's *measured residual failure distribution*. The paper demonstrates downstream utility but does not close the loop from student errors back into SQL generation allocation.

**Follow-up/prior-art check.** SAC-SQL (Findings EMNLP 2025) already performs synthetic-data training with a structure-aware easy-to-hard curriculum, so `generate harder SQL` or generic curriculum is not novel. The viable delta is **closed-loop error-targeted generation under a fixed synthetic-data budget**, using held-out target-model failures rather than a static complexity score.

---

## A5. MTSQL-R1 — ACL 2026 Main

**Paper.** Taicheng Guo, Hai Wang, Chaochun Liu, Mohsen Golalikhani, Xin Chen, Xiangliang Zhang, Chandan K. Reddy. *MTSQL-R1: Towards Long-Horizon Multi-Turn Text-to-SQL via Agentic Training*. ACL 2026 Main.  
Paper: https://aclanthology.org/2026.acl-long.1563/  
Code/models: https://github.com/taichengguo/MTSQL-R1

**Task.** Long-horizon multi-turn Text-to-SQL on CoSQL and SParC.

**Core method.** MTSQL-R1 casts the task as an MDP with database execution and persistent dialogue memory, trains a self-taught warm-start model, then curriculum RL with outcome/process rewards over propose→execute→verify→refine trajectories. With Qwen3-4B it reports 79.9/79.0 EX on CoSQL/SParC vs the strongest Short-Horizon RL baseline at 75.2/75.8. Under the more realistic `Predicted as Prior` CoSQL evaluation, MTSQL-R1 drops from 79.9 (gold prior) to 76.5, while Direct RL drops from 75.2 to 71.2.

**Reproducible boundary.** The default multi-turn setting populates history with ground-truth SQL; the paper explicitly evaluates model-generated prior SQL as the more faithful deployment setting and observes a remaining gap. The memory tool description itself retrieves historical questions and ground-truth SQL during the standard setup.

**Follow-up/prior-art check.** Rose-SQL (ACL 2026) already introduces structured Role-State evolution, so `better structured memory` is crowded. Classic scheduled sampling also makes generic exposure-bias correction incremental. The narrower viable delta is **training on naturally model-generated executable-wrong histories, with targeted recovery curriculum and matched RL token budget**, while leaving memory architecture unchanged.

---

# 排名 1：Constraint-Conditioned MDD SQL Selector

- **Anchor Paper:** DPC, ACL 2026 Main (Li et al.). Public paper and official code above.
- **Problem:** DPC may select a winner using a distinguishing synthetic world that violates the target application's constraints. The proposed method asks a narrower method question: *does candidate selection improve when an MDD must be both discriminative and realizable under trusted target constraints, and when the selector is allowed to return UNRESOLVED if no valid duel exists within budget?*
- **Input / Intermediate Object / Output:**
  - Input: natural-language question `q`, schema `S`, trusted hard constraints `Γ` (PK/FK/UNIQUE/CHECK plus explicitly whitelisted dbt/domain invariants), and `K` SQL candidates.
  - Intermediate: champion/challenger pair plus a constraint-valid MDD `D_Γ` satisfying `D_Γ ⊨ Γ` and `Exec(y_c,D_Γ) != Exec(y_h,D_Γ)`; otherwise `UNRESOLVED`.
  - Output: ranked candidates / top-1 SQL.

### Exact Delta

| 维度 | Anchor Paper: DPC | 本 idea |
|---|---|---|
| 输入表示 | q + schema + K candidates | q + schema + K candidates + trusted constraint set Γ |
| 中间表示 | context-fitting, discriminative MDD | **constraint-valid discriminative MDD** + violation trace + UNRESOLVED state |
| 训练目标 | 无训练 | 无训练 |
| 推理过程 | LLM Slicer/Tester synthesizes MDD, Python Solver selects | Tester proposal → deterministic DBMS/SMT constraint validation → bounded repair/search → DPC Solver; fallback on UNRESOLVED |
| 监督信号 | execution divergence + Python consistency | same, but only after all hard constraints pass |
| 评价指标 | candidate-selection EX | **Candidate Selection Top-1 EX on a preregistered constraint-sensitive subset** |

**Why not a module swap.** The decision rule itself changes: an invalid distinguishing world is no longer admissible evidence, and candidate duels can remain unresolved. Old constraint solvers are implementation tools/baselines, not the claimed contribution.

- **Primary Metric:** `Top1-EX_cs = (1/N_cs) Σ_i 1[Exec(ŷ_i,D_i)=Exec(y*_i,D_i)]`, evaluated on an outcome-blind, preregistered constraint-sensitive subset `D_cs`.
- **Secondary Metrics:** MDD constraint-violation rate; Candidate MRR; selector latency; selector token cost.
- **Core Hypothesis:** On BIRD/Spider, with Qwen2.5-Coder-7B, `K=5`, and at most 3 MDD repair rounds, Constraint-Conditioned MDD improves `Top1-EX_cs` by **≥4.0 absolute points** over original DPC while selection latency is **≤1.5×** DPC.
- **Strongest Baselines:** original DPC; Self-Consistency; Execution-Guided selection; DDL-only post-filter / constraint-aware distinguishing-DB baseline implemented with the same solver budget.
- **Minimal 20–50 Example Test:** 40 BIRD Mini-Dev/Dev candidate duels selected *without seeing system outcomes*: multi-table cases with explicit PK/FK/UNIQUE/CHECK and candidate disagreement in join/null/duplicate/group logic. Compare original DPC, simple DDL post-filter, proposed iterative constrained MDD, and Self-Consistency. Success signal: ≥3 net additional correct selections over DPC (7.5 points on 40) **and** ≥50% reduction in invalid MDDs. Kill if ≤1 net additional correct, if invalid MDDs are already negligible, or if a one-pass DDL post-filter matches the proposed method.
- **Full Experiment:** BIRD + Spider; at least four baselines above; cross-database held-out reporting; separate DDL-only vs richer trusted-invariant strata. Report paired bootstrap 95% CIs for EX deltas and McNemar tests for paired top-1 correctness.
- **Ablations:** (1) remove hard-constraint validator; (2) DDL-only Γ vs DDL+trusted business invariants; (3) remove UNRESOLVED fallback and force a winner.
- **Kill Criterion:** no ≥4-point gain on constraint-sensitive cases; no downstream gain despite lower invalid-MDD rate; trivial post-filter equals method; effect appears in only one database; follow-up work found to already implement realizability-conditioned DPC.
- **Novelty Risk:** medium-low. Constraint-aware test generation itself is old; novelty survives only if realizability changes modern LLM candidate-selection decisions materially.
- **Feasibility:** high; official DPC code, BIRD/Spider, no model training.
- **Final Scores (1–5):** Anchor Strength **5**; Quantifiability **5**; Novelty **4**; Feasibility **4**; Reproduction Cost **2** (1=low, 5=high); Incremental-Risk **2** (1=low, 5=high).
- **Two-Sentence Pitch:** DPC makes SQL verification executable by inventing a distinguishing world, but that world is not required to be possible under the target application's constraints. We make realizability part of the selector's evidence semantics and test whether rejecting impossible counterexamples measurably improves top-1 SQL selection.

---

# 排名 2：Risk-Adaptive Execution Checkpointing

- **Anchor Paper:** VET, Findings ACL 2026 (Wang et al.). Public paper and fully specified public-benchmark protocol above.
- **Problem:** VET executes every reasoning step, which is accurate but latency-heavy. The method question is not generic routing: *which specific VET steps are causally worth executing under a fixed database-call budget?*
- **Input / Intermediate Object / Output:**
  - Input: question, schema, database, current VET trace state, proposed next executable operation `o_t`.
  - Intermediate: calibrated criticality `r_t=P(final error if checkpoint t is skipped | pre-execution features)`.
  - Output: EXECUTE/SKIP decision per intermediate checkpoint and final SQL.

### Exact Delta

| 维度 | Anchor Paper: VET | 本 idea |
|---|---|---|
| 输入表示 | q + candidate schema + full trace | same + pre-execution step features and remaining call budget |
| 中间表示 | every Python step is executed | **step criticality score** + remaining-budget state |
| 训练目标 | training-free | lightweight calibrated classifier: weighted BCE + call-cost regularization |
| 推理过程 | execute every step, repair if needed | execute only if `r_t≥τ(B)`; mandatory final execution; otherwise continue trace without DB call |
| 监督信号 | runtime execution feedback | offline leave-one-checkpoint-out / repair-trigger labels from logged traces |
| 评价指标 | overall EX | **Execution Accuracy at fixed mean intermediate DB-call budget (EX@B)** |

**Why not generic budget routing.** BAP-SQL routes/rewrites observations at query level. This method leaves VET's trace generator fixed and learns the *marginal necessity of each checkpoint* from causal replay labels.

- **Primary Metric:** `EX@B = (1/N)Σ_i Correct_i`, subject to `(1/N)Σ_i intermediate_DB_calls_i ≤ B`. Main setting: `B = 0.5 ×` full-VET mean intermediate calls on the same evaluation set.
- **Secondary Metrics:** mean DB calls/query; latency; token cost; unconstrained/full-budget EX.
- **Core Hypothesis:** On BIRD Moderate+Challenging with Qwen2.5-Coder-14B, at **50% of full-VET intermediate DB-call budget**, Risk-Adaptive Checkpointing improves `EX@B` by **≥4 points** over the strongest same-budget fixed/query-level gating baseline, remains within **1.5 points** of full VET, and reduces latency to **≤0.8×** full VET.
- **Strongest Baselines:** full VET; final-only/no intermediate execution; every-other-step / fixed operator checkpoints; random 50%-budget checkpoints; BAP-style query-level `full VET vs final-only` gate matched to the same mean DB-call budget.
- **Minimal 20–50 Example Test:** 50 BIRD Moderate/Challenging examples with vanilla VET traces of ≥3 steps, selected outcome-blind. Use 25 examples only to fit/calibrate the tiny checkpoint model and 25 locked examples for the kill test. Success: on the locked 25, ≥2 additional correct tasks vs strongest 50%-budget baseline and no more than one task lost vs full VET; kill if improvement <1 task, an operator-only heuristic matches it, or ≥80% of checkpoints are empirically indispensable.
- **Full Experiment:** BIRD + Spider 2.0-lite (plus Spider-DK as robustness split if budget permits); DeepSeek-Chat and Qwen2.5-14B; at least the five baselines above. Report accuracy-cost Pareto curves, paired bootstrap 95% CIs, McNemar for EX, and paired latency CIs.
- **Ablations:** (1) operator-only risk vs learned risk; (2) remove dynamic trace features (prior repairs/cardinality uncertainty); (3) fixed threshold vs budget-calibrated threshold.
- **Kill Criterion:** Pareto frontier is not improved; full VET needs nearly all checkpoints; simple fixed checkpoints equal the learned policy; gains vanish on Spider2-Lite; DB calls fall but EX drops more than the preregistered tolerance.
- **Novelty Risk:** medium. Active probing/budget control is crowded, so the paper must stay on *step-level causal checkpoint value inside VET*.
- **Feasibility:** high-medium; no large-model training, but requires reproducible VET trace generation/replay.
- **Final Scores:** Anchor Strength **5**; Quantifiability **5**; Novelty **4**; Feasibility **4**; Reproduction Cost **2**; Incremental-Risk **3**.
- **Two-Sentence Pitch:** VET buys accuracy by executing every reasoning step, but its own limitation is that every step waits on the database. We learn which checkpoints actually prevent final SQL errors and optimize EX under a hard DB-call budget, rather than adding another global router.

---

# 排名 3：Contrastive Value-Link Probe Selector

- **Anchor Paper:** DIVER, SIGMOD 2026 (Nan et al.), with official code.
- **Problem:** DIVER's dynamic tools are useful, but open-ended LLM routing is not explicitly optimized to distinguish competing value-link hypotheses; its tool-route ablation does not reduce EX. The proposed selector chooses probes by *expected discrimination among concrete value-link candidates*.
- **Input / Intermediate Object / Output:**
  - Input: question, schema, database, unresolved value mention(s), max probe budget `B`.
  - Intermediate: candidate hypotheses `H={h_j=(column, normalized_value, operator)}` with calibrated priors; for each legal probe, an estimated partition of possible outcomes and expected posterior entropy.
  - Output: verified value-link set, then final SQL through unchanged DIVER evidence/SQL generation.

### Exact Delta

| 维度 | Anchor Paper: DIVER | 本 idea |
|---|---|---|
| 输入表示 | clause + CoTF + toolbox | same + explicit top-H value-link hypothesis set and hard probe budget |
| 中间表示 | free-form thoughts/facts | **posterior over link hypotheses** + expected information gain per probe |
| 训练目标 | training-free LLM routing | small calibration model for hypothesis priors/outcome likelihoods; no SQL-generator training |
| 推理过程 | LLM chooses next tool from toolbox | choose `argmax_a IG(H;a)/(cost(a)+ε)`, update posterior, adaptive stop |
| 监督信号 | observed tool facts | gold value-link labels for calibration; actual probe outcomes for posterior update |
| 评价指标 | schema/value F1 + EX/VES | **Value Linking F1 under ≤3 probes** |

**Why not another probing agent.** The action space remains DIVER's toolbox; the innovation is a computable value-link discrimination objective and adaptive stopping against an explicit hypothesis set.

- **Primary Metric:** `Value-F1@3 = 2PR/(P+R)` over predicted vs gold value-link entities, with at most 3 DB probes per question.
- **Secondary Metrics:** downstream SQL EX; DB calls/query; token cost; latency.
- **Core Hypothesis:** On BIRD value-ambiguous Moderate/Challenging examples with SFT-CodeS-3B and `B=3`, the method improves `Value-F1@3` by **≥4.0 points** over original DIVER tool selection under the same probe cap, while downstream EX improves by **≥2 points** and token cost increases by **≤10%**.
- **Strongest Baselines:** original DIVER capped at 3 probes; CodeS value linking; fixed `uniq_value→sim_value_in→sim_columns` rule; random DIVER probe; PV-SQL/SDE-SQL policy when protocol-compatible; BAP-style budget-aware observation policy as a high-pressure follow-up baseline.
- **Minimal 20–50 Example Test:** 40 BIRD examples selected by two preregistered, outcome-blind criteria: non-empty gold value-link set and at least two plausible retrieved link hypotheses before probing. Stratify 20 Moderate/20 Challenging. Success: ≥7 F1-point signal on the 40-example set, ≤3 calls, and no downstream EX decrease; kill if <3 F1 points, a fixed tool order matches it, or corrected value links do not translate into any downstream SQL benefit.
- **Full Experiment:** BIRD + DR.Spider/Spider-DK content/linking perturbation splits; at least four baselines above; cross-database test split; report 95% paired bootstrap CIs for F1/EX and Pareto curves for F1/EX vs calls/tokens.
- **Ablations:** (1) remove information gain and choose highest-prior hypothesis probe; (2) remove cost normalization; (3) disable adaptive stop and always spend all 3 calls.
- **Kill Criterion:** no F1@budget gain; fixed heuristic equals method; gains only on BIRD but not perturbation/cross-DB splits; better linking fails to improve SQL EX; newest probing follow-up already performs explicit link-hypothesis information-gain selection.
- **Novelty Risk:** medium-high because probing is crowded; novelty is only defensible if the hypothesis-discrimination objective gives a measurable F1/cost advantage over DIVER/BAP/PV-SQL-style baselines.
- **Feasibility:** high; official DIVER code, no end-to-end retraining required.
- **Final Scores:** Anchor Strength **5**; Quantifiability **5**; Novelty **3.5**; Feasibility **4**; Reproduction Cost **2**; Incremental-Risk **3**.
- **Two-Sentence Pitch:** DIVER proves that interactive database probing helps value linking, but its route is not directly trained or optimized to eliminate competing link hypotheses. We turn probing into a budgeted discrimination problem and test a primary metric—Value Linking F1@3—that can fail independently of downstream model rhetoric.

---

# 排名 4：Error-Targeted SQL Synthesis Scheduler

- **Anchor Paper:** SQL-Factory, PVLDB 19(3), 2025 (VLDB 2026 cycle), Li et al., with public code/artifacts.
- **Problem:** SQL-Factory allocates generation toward structurally rich/under-covered schema regions, while the target Text-to-SQL student may already solve those regions and fail elsewhere. The method closes the loop by allocating a fixed synthesis budget to *measured residual error signatures of the target student on disjoint calibration databases*.
- **Input / Intermediate Object / Output:**
  - Input: database schemas, fixed synthetic budget `G`, target student model `M`, small real calibration split disjoint from final test DBs.
  - Intermediate: error-need distribution over deterministic SQL signatures (missing/wrong join edge, GROUP/HAVING, DISTINCT, nesting, set op, window, value predicate, etc.) and schema motifs.
  - Output: a generated `<NL,SQL>` training corpus of exactly `G` examples, then fine-tuned student `M'`.

### Exact Delta

| 维度 | Anchor Paper: SQL-Factory | 本 idea |
|---|---|---|
| 输入表示 | schema + SQL Pool statistics | schema + SQL Pool + target-student error-need vector from disjoint calibration DBs |
| 中间表示 | schema coverage, quality/diversity statistics | **error-coverage deficit × schema motif** plus existing coverage stats |
| 训练目标 | generate diverse/high-quality SQL | allocate fixed G to maximize coverage of unresolved student failure signatures; student training budget held fixed |
| 推理过程 | Management Team switches generation/expansion from coverage/quality | scheduler score `β·coverage_gap + (1-β)·expected_error_coverage`; at most one closed-loop rescore |
| 监督信号 | SQL quality/diversity/schema coverage | deterministic AST/plan diff between calibration predictions and gold SQL + execution correctness |
| 评价指标 | diversity + downstream EX | **held-out-database EX after equal-G/equal-training-token synthesis** |

**Why not generic curriculum/hard mining.** SAC-SQL already orders synthetic examples by static structural difficulty. This method conditions *where new SQL is generated* on the deployed student's observed, typed residual errors and tests on disjoint schemas under an exactly matched generation/training budget.

- **Primary Metric:** `EX_G = (1/N_test)Σ_i 1[Exec(M'_G(q_i),D_i)=gold_i]`, where every synthesis method gets exactly `G` generated pairs and the student gets the same fine-tuning tokens/steps.
- **Secondary Metrics:** hard-structure EX; generated-set error-signature coverage; generation API cost; student training tokens.
- **Core Hypothesis:** On Spider and BIRD with Qwen2.5-Coder-1.5B, `G=20K` synthetic pairs and matched fine-tuning tokens, Error-Targeted Scheduling improves held-out-database `EX_G` by **≥2.0 points** over original SQL-Factory and **≥1.0 point** over a SAC-SQL-style static structure curriculum, with generation cost **≤1.1×** SQL-Factory.
- **Strongest Baselines:** original SQL-Factory scheduler; SAC-SQL-style structure-aware curriculum/sampling; SQL-Factory with uniform/random error-category reweighting; official-data SFT; OmniSQL/other generation baseline where artifacts permit.
- **Minimal 20–50 Example Test:** use 3–5 training/calibration DBs and 3 held-out DBs; lock 40 held-out questions stratified by join/nesting/aggregation. Generate only `G=2K` examples per scheduler using the same SQL-Factory API/local-model budget; LoRA-tune Qwen2.5-Coder-1.5B for one matched epoch. Success: ≥2 net additional correct questions (5 points on 40) over original SQL-Factory **and** improved accuracy specifically on the targeted error signatures; kill if ≤1 net correct, static structure curriculum matches it, or the benefit disappears on held-out DBs.
- **Full Experiment:** Spider + BIRD with database-disjoint calibration/test partitions; Qwen2.5-Coder-1.5B and at least one second small backbone; >=4 baselines above; 3 random seeds; paired bootstrap/McNemar on final EX and seed-level CI on training variation.
- **Ablations:** (1) remove student feedback → original schema-coverage scheduler; (2) remove the closed-loop rescore after first small training round; (3) keep error categories but make their weights uniform.
- **Kill Criterion:** no ≥2-point EX gain under equal G; gains explained entirely by more training tokens/API cost; static SAC-SQL curriculum equals method; targeted signatures increase in corpus but held-out EX does not; effect only on one benchmark.
- **Novelty Risk:** medium-high. Curriculum/hard-example mining is old; novelty depends on closed-loop *generation allocation from target-model residual error signatures* and strict held-out-schema evidence.
- **Feasibility:** medium; artifacts are public but even the kill test requires data generation plus a small student fine-tune.
- **Final Scores:** Anchor Strength **5**; Quantifiability **5**; Novelty **3.5**; Feasibility **3**; Reproduction Cost **4**; Incremental-Risk **3**.
- **Two-Sentence Pitch:** SQL-Factory optimizes a generic SQL corpus; our scheduler asks what SQL this particular student still needs. Under equal synthetic examples and equal training tokens, the idea succeeds only if model-error-conditioned generation beats both SQL-Factory's coverage scheduler and a strong static structure curriculum on unseen databases.

---

# 排名 5：Predicted-History Curriculum Training

- **Anchor Paper:** MTSQL-R1, ACL 2026 Main (Guo et al.), with official code/models.
- **Problem:** MTSQL-R1's standard history contains gold prior SQL, but deployment history contains the model's own predictions; the paper reports 79.9 EX with gold prior vs 76.5 with predicted prior on CoSQL. The proposed method changes the *training state distribution*, not the memory architecture.
- **Input / Intermediate Object / Output:**
  - Input: dialogue prefix containing naturally model-generated prior SQL, current utterance, schema/database.
  - Intermediate: mixed history state `H_p` where previous turns come from gold or current-checkpoint rollouts; predicted histories are typed as correct / execution-error / executable-wrong.
  - Output: current-turn SQL through the unchanged MTSQL-R1 agent/tool interface.

### Exact Delta

| 维度 | Anchor Paper: MTSQL-R1 | 本 idea |
|---|---|---|
| 输入表示 | standard training/eval history dominated by gold prior SQL | curriculum mixture of gold and **naturally model-generated prior SQL** |
| 中间表示 | persistent question+SQL memory | same memory; add history-source/error-type tag only for sampler/training bookkeeping |
| 训练目标 | original outcome + process RL rewards | same rewards + state-distribution curriculum; optional recovery-efficiency cost on noisy-history states |
| 推理过程 | propose→execute→verify→refine | unchanged at inference |
| 监督信号 | gold/history environment outcomes | on-policy rollout histories; oversample naturally occurring executable-wrong priors |
| 评价指标 | gold-prior and predicted-prior EX | **Predicted-History Execution Accuracy** |

**Why not another memory module.** Rose-SQL already covers structured historical state. This idea leaves retrieval, memory representation, tools, and inference architecture unchanged; only the training state distribution is made deployment-faithful and targeted to real executable-wrong priors.

- **Primary Metric:** `PH-EX = (1/N)Σ_i 1[Exec(ŷ_i,D_i)=gold_i]` where *all preceding SQL in the dialogue is generated by the same deployed model*, never replaced by gold SQL.
- **Secondary Metrics:** gold-history EX; DB/tool calls; latency; Exact Match.
- **Core Hypothesis:** On CoSQL with Qwen3-4B and the same RL token budget as MTSQL-R1, Predicted-History Curriculum raises predicted-prior EX from the anchor's **76.5 to ≥79.0** (+2.5 points), while gold-prior EX decreases by **≤0.5 point** and average tool calls rise by **≤10%**.
- **Strongest Baselines:** released MTSQL-R1; Direct RL; Short-Horizon RL; naive uniform scheduled sampling over all predicted histories; Rose-SQL when inference/training protocol can be matched.
- **Minimal 20–50 Example Test:** identify 40 CoSQL current turns where a released/current checkpoint naturally produces at least one executable-wrong prior SQL before the target turn—no hand-crafted corruption. Build ~200 training states from disjoint training dialogues with the same natural predicted-history process; run a small LoRA/RL continuation with fixed tokens. Success: ≥2 net additional correct target turns over released MTSQL-R1 and clean/gold-history regression ≤1 turn; kill if <1 net improvement, uniform scheduled sampling matches it, or the effect vanishes when evaluated on complete predicted-history dialogues rather than the selected error subset.
- **Full Experiment:** CoSQL + SParC; Qwen3-4B + Llama3.2-3B; at least MTSQL-R1, Direct/Short-Horizon RL, uniform scheduled sampling, and Rose-SQL if compatible; full-dialogue predicted-history rollouts; three seeds; paired bootstrap/McNemar for EX and seed-level CIs.
- **Ablations:** (1) uniform predicted-history sampling vs executable-wrong prioritization; (2) fixed predicted-history mixture `p` vs curriculum `p_k`; (3) remove recovery-efficiency penalty while keeping state distribution.
- **Kill Criterion:** naive scheduled sampling is equal; predicted-history EX improves <2.5 points; gold-history performance drops >0.5 point; improvement is only on selected injected/error cases rather than full rollout; a 2026 follow-up is found to already train on self-generated erroneous dialogue histories.
- **Novelty Risk:** high relative to the other four because exposure-bias/scheduled-sampling ideas are old and Rose-SQL already pressures the history side. It survives only if prioritizing *naturally executable-wrong SQL histories inside long-horizon agentic RL* materially beats uniform scheduled sampling.
- **Feasibility:** medium; official MTSQL code/models exist, but RL/rollout reproduction is costlier.
- **Final Scores:** Anchor Strength **5**; Quantifiability **5**; Novelty **3**; Feasibility **3**; Reproduction Cost **4**; Incremental-Risk **4**.
- **Two-Sentence Pitch:** MTSQL-R1 already shows that model-generated history is the deployment-faithful setting and that accuracy drops there. We train on the model's own naturally occurring executable-wrong history states, without changing memory architecture, and require a measured predicted-history EX gain over both released MTSQL-R1 and naive scheduled sampling.

---

# Screening decisions — not admitted to the final five

The following otherwise-interesting directions were excluded because they violate the method-only or novelty constraints:

- **SampledRewriteUnsoundness / ReSequel:** useful failure probe, but the obvious repair collapses toward established query-equivalence/test-suite/counterexample validation machinery unless a genuinely new algorithmic object is found. **analysis-only for now.**
- **Semantic Repeatability Gap / SemBench:** strong evaluation question but currently primarily an analysis/metric paper. **analysis-only.**
- **DownstreamImpact-GEM / GLEAM:** pairwise F1 vs downstream analytical truth is a valuable evaluation mismatch, but a nontrivial method beyond cost-sensitive matching is not yet sufficiently separated from existing entity-resolution objectives. **analysis-only until a method delta is identified.**
- **OpenSQL supervision fanout:** the failure is plausible, but `validate every augmented label with another verifier/test-suite` is presently a module recombination rather than a clean novel method. Excluded.
- **Large-schema pruning / JoinBridge Blind Spot:** promising, but the referenced 2026 large-schema artifact was not fully available in the prior pass; under the current rules it stays BLOCKED rather than being replaced by synthetic-only evidence.

# Recommended order before implementation

1. Run the 40-example **Constraint-Conditioned DPC** kill test first: lowest cost and cleanest causal delta.
2. Run the 50-example **VET checkpoint-value** replay/calibration test second.
3. Run the 40-example **DIVER value-link probe** test third.
4. Only if more expensive training is justified, run the small-budget **SQL-Factory scheduler** pilot.
5. Run **MTSQL predicted-history curriculum** last because it has the highest reproduction cost and incremental-risk.

No method should proceed to a full implementation if its preregistered kill condition fires.