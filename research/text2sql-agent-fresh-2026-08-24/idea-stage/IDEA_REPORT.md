# Fresh Idea Discovery Report — Text2SQL × Database/Data Agents

**Date:** 2026-08-24  
**Branch:** `research/text2sql-agent-fresh-2026-08-24`  
**Pipeline:** research-lit → batch idea generation → novelty screening → adversarial review → refinement  
**Independence rule:** started from `main`; this run does not use earlier Text2SQL idea rankings or conclusions.  
**Empirical pilots:** NOT RUN.  
**Formal external reviewer receipt:** UNAVAILABLE in this connector session; novelty/review assessments are provisional.

---

## Executive Summary

A fresh 2025–2026 literature pass shows that many formerly attractive Text2SQL ideas are now saturated: multi-turn RL, adaptive turn budgets, database probing, semantic memory, semantic IR/layers, candidate voting, execution verification, clarification, query repair, unknown-database routing, and AI-native SQL optimization all have close recent work.

The useful research frontier therefore shifts from "add another agent module" toward questions about **what evidence the agent observes, what it is epistemically allowed to claim, how agent policies transfer, and how persistent experience behaves when the environment changes**.

This run generated **36 raw candidates**, ran two novelty passes, and retains **12 serious candidates**. The current top five are:

1. **EvidenceSketch-SQL** — optimize the *representation* of database observations, not only which probe to run.
2. **Schema-Evolution × Memory Poisoning** — measure when verified Text2SQL memory becomes harmful after schema/business drift.
3. **CausalGuard-SQL** — require causal identifiability evidence before a data agent turns relational observations into causal/recommendational claims.
4. **HarnessTransfer-SQL** — measure whether orchestration strategies themselves overfit benchmarks/harnesses.
5. **AutonomyOracle-SQL** — learn the cheapest sufficient autonomy regime using counterfactual forced-regime outcomes.

A second novelty pass materially downgraded three initially strong ideas:

- **PoU-SQL** now has to beat structural abstention (`Never the Number`, Aug 2026), LatentRefusal, and other no-answer systems; only a machine-checkable minimal-missing-information certificate remains interesting.
- **GoldChallenge-SQL** now has to beat SAR-Agent annotation-error detection and GBV-SQL's Gold Error audit; only an online *contestable-oracle protocol* remains defensible.
- **CrossDB-Key Agent** is no longer a strong generic method claim because DAB explicitly benchmarks dirty cross-database joins and production systems already report deterministic join-inference machinery. A focused benchmark/audit angle may still be useful.

---

# 1. Fresh Literature Landscape

## 1.1 Autonomy is now a spectrum, not a binary

**Agentic-SQL Revisited** (arXiv:2608.15389) organizes Text2SQL along an autonomy axis: constrained, in-context, iterative, agentic, and reasoning-internalized. Its synthesis reports uneven transfer across Spider/BIRD/Spider2 and non-trivial costs from greater autonomy.

**SQL-Trail** and **MTSQL-R1** (ACL 2026) both train multi-turn database interaction. SQL-Trail uses interleaved execution feedback and adaptive turn budgets; MTSQL-R1 casts multi-turn Text2SQL as an MDP with propose→execute→verify→refine and persistent dialogue memory.

**BAP-SQL** (arXiv:2608.02876) makes observation planning budget-aware; gains are strongest under tight budgets and attenuate with larger budgets/models.

**AGENTIQL** already routes between a modular agentic pipeline and a simpler parser.

**Implication:** generic "route hard questions to an agent" or "adapt turns to difficulty" is crowded. A remaining question is whether the **minimum qualitatively sufficient autonomy regime** is a learnable counterfactual variable distinct from risk score/token budget.

---

## 1.2 Probing is crowded; observation representation is less studied

**SDE-SQL** (ACL 2026) performs self-driven SQL probes. **PV-SQL** probes concrete records to resolve value/column/join ambiguity. **PExA** uses parallel atomic SQL tests inspired by software testing. **BAP-SQL** decides how to form observations under budget.

Most of these systems optimize **which action/probe to run**. They usually still return rows/text to the LLM. That leaves a sharper interface question:

> For a given uncertainty, what is the smallest structured observation that preserves the evidence needed for the next decision?

Possible database-native evidence units include row count, distinct/key coverage, join match rate, null fraction, heavy hitters, quantiles, temporal bounds, many-to-many risk, uniqueness witnesses, and compact lineage/provenance fingerprints.

---

## 1.3 Candidate selection, verification, and repair are highly occupied

**R³-SQL** groups candidates by execution result, ranks result groups, and resamples when needed. **DPC** builds a Minimal Distinguishing Database and cross-checks SQL with Python/Pandas. **VET** executes stepwise semantics on the real database. **SafeQL** restricts refinement to a safe query space. **GBV-SQL** performs SQL2Text back-translation validation. Recent repair work also targets executable-but-wrong SQL.

**Implication:** multiple candidates + judge, execution reflection, counterexample DBs, and targeted clause repair should be baselines/components rather than paper novelty.

---

## 1.4 Text2SQL memory is now a mature sub-direction

**AgentSM** stores interpretable structured programs derived from execution traces and reports lower token/trajectory cost plus stronger Spider2-Lite performance.

**Crystallization in Text-to-SQL** (arXiv:2608.07213) separates replay, retention, and held-out same-database transfer; verified corrected queries improve held-out first-attempt performance.

**MERIT**, **SDAM**, **GATE**, and **EvoSQL** cover dynamic retrieval, contradiction-aware/evolving memories, execution-grounded semantic memory, and memory-augmented generator/critic loops.

General 2026 agent-memory work further covers portability, governed memory, lifecycle operations, and model-family transfer.

**Implication:** basic success/failure memory, phase-aware retrieval, portable memory, or "store verified SQL" are no longer good standalone claims. More interesting are **negative transfer under environmental change** and **compatibility when the backbone changes**.

---

## 1.5 Semantic planning/layers increasingly separate probabilistic grounding from deterministic execution

**Bounded Semantic Planning and Deterministic Compilation** (arXiv:2608.16663) uses stochastic semantic choices plus deterministic graph traversal/grain lowering/SQL compilation/checks.

**SemPlan** (arXiv:2608.13612) compares direct SQL, bounded agents, semantic request/planning, and clarification/state planning; structured planning changes the failure profile but does not dominate every cost/policy dimension.

A **Semantic-Layer-Mediated Agent** (arXiv:2606.31041) uses a curated semantic layer and compact semantic query IR before deterministic dialect compilation; **SemanticAgent** targets semantic validity during synthesis/diagnosis.

**Implication:** semantic IR + compiler and semantic layer + SQL lowering are crowded. A result-contract representation may still be useful inside another idea but is weak as a standalone paper.

---

## 1.6 Unanswerability/refusal is more crowded than first-pass search suggested

**ABISS** (arXiv:2607.23340) unifies ambiguous/unanswerable categories. **Query Carefully** explicitly detects no-answer questions. **LatentRefusal** (ACL 2026) predicts answerability from hidden activations as an attachable safety layer.

Most importantly for the freshest search, **Never the Number: Structural Abstention for AI Systems Whose Answers Are Consumed as Fact** (arXiv:2608.13926, Aug 14 2026) proposes a trusted deterministic kernel where unanswerable requests are *unrepresentable* rather than confidence-rejected.

**Implication:** an idea cannot claim novelty from refusal, answerability gating, or structural abstention alone. The remaining PoU angle must produce a **checkable certificate of the minimal missing fact/source/definition** plus a recovery test.

---

## 1.7 Benchmark gold quality is actively being repaired already

**Pervasive Annotation Errors Break Text-to-SQL Benchmarks and Leaderboards** (arXiv:2601.08778) reports large annotation-error rates in audited benchmark subsets and large effects on system scores/ranks. Its released **SAR-Agent** actively interacts with the database to detect annotation errors.

**ReViSQL** shows verified training data can materially improve the same RL procedure. **GBV-SQL** (ACL 2026) includes a Gold Error typology/audit alongside semantic validation.

**Implication:** "detect wrong gold SQL" is already occupied. A remaining systems/evaluation idea is to make the oracle **contestable during training/evaluation**: maintain versioned label confidence, route disagreements to verification, allow `model_wrong / gold_wrong / both_valid / unresolved`, and quantify human-audit savings/reward stability.

---

## 1.8 Schema evolution is benchmarked, but memory under evolution is still a clean intersection

**EvoSchema** (arXiv:2603.10697) introduces ten schema-evolution perturbation types and shows table-level changes are especially damaging to Text2SQL.

Memory papers generally evaluate reuse in a stable database setting. General engineering discussions on agent memory migrations exist, but the specific question remains experimentally sharp:

> What happens when previously verified Text2SQL memory survives across controlled schema or business-semantic evolution?

This creates a measurable negative-transfer problem rather than another memory architecture.

---

## 1.9 Broader data-agent benchmarks expose problems outside single-database SQL

**Data Agent Benchmark (DAB)** evaluates multi-database integration, ill-formatted join keys, unstructured-text transformation, and domain knowledge. **DataSpace**, **FDABench**, and **AgenticDataBench** broaden the setting further to heterogeneous files/artifacts/workflows.

DAB makes dirty-key cross-database joins an explicit challenge. Production data-agent systems also publicly describe deterministic join-inference helpers based on sampled values and normalization rules.

**Implication:** a generic "agent reasons about dirty cross-DB keys" idea is too weak. A focused benchmark on **join-evidence sufficiency/false-join safety** may survive, but it is no longer a top method bet.

---

## 1.10 Business/data agents increasingly exceed descriptive SQL

**CORGI** explicitly includes descriptive, explanatory, predictive, and recommendational business questions and notes the need for causal reasoning, forecasting, and strategy.

This exposes a safety gap: an agent can execute perfectly valid descriptive SQL and then make an unsupported causal/recommendational statement in natural language.

The promising research variable is not generic causal discovery. It is the **boundary between what relational observations justify and what the final claim asserts**.

---

## 1.11 AI-native SQL is a fast-moving optimizer/reliability frontier

**Spider 2.0-AIFunc** benchmarks native AI functions inside SQL. **Larch**, **Stretto**, **Horrila**, and **SAGE** study optimization, algebra, quality/cost tradeoffs, and execution strategies for semantic/AI operators.

**Implication:** generic AI-SQL optimization is crowded. Potentially open questions are upstream/downstream of physical planning: **is probabilistic AI computation necessary at all?** and **how should correctness be defined statistically when SQL contains nondeterministic AI operators?**

---

# 2. Directions Explicitly Killed as Standalone Novelty

| Direction | Status | Why |
|---|---|---|
| Active schema/database probing | KILL | SDE-SQL, PV-SQL, PExA, BAP-SQL, AutoLink |
| Generic multi-turn RL | KILL | SQL-Trail, MTSQL-R1, ReEx-SQL |
| Adaptive turn count/budget | KILL | SQL-Trail, BAP-SQL |
| Planner/critic/verifier agent | KILL | many recent systems |
| Multi-candidate voting/ranking | KILL | R³-SQL, DPC, others |
| Generic SQL repair | KILL | SafeQL, BIRD-CRITIC/SWE-SQL, recent repair work |
| Basic Text2SQL memory | KILL | AgentSM, GATE, Crystallization, MERIT, SDAM |
| Semantic IR/compiler | KILL | semantic-layer agents, bounded semantic planning |
| Generic clarification/EIG | KILL | ABISS, PRACTIQ, EIG interactive SQL |
| Minimal clause patch after feedback | KILL | STEPS already close |
| Unknown-database selection | KILL | MDB-Link/TACO-type settings |
| Generic benchmark error detection | KILL | SAR-Agent, GBV-SQL audit |
| Generic no-answer refusal | KILL | LatentRefusal, Query Carefully, structural abstention |
| Generic dirty-key cross-DB join agent | DOWNGRADE | DAB + existing deterministic join tooling |
| Generic claim provenance | KILL | TRACER/general tool-agent provenance |
| Agentic GPU/workload scheduling | KILL | HEXGEN-TEXT2SQL |
| Generic AI-SQL optimizer | KILL | Larch, Stretto, Horrila, SAGE |

---

# 3. Raw Idea Bank — 36 Candidates

| # | Idea | Primary variable | After novelty pass |
|---:|---|---|---|
| 1 | **EvidenceSketch-SQL** | DB observation representation | SURVIVE |
| 2 | Privacy-Aware Evidence Sketches | evidence under row-exposure/privacy budget | SURVIVE as variant |
| 3 | PoU-SQL | minimal missing-information certificate | SURVIVE, narrowed |
| 4 | Ambiguity Proof Object | evidence showing exactly what clarification resolved | MEDIUM |
| 5 | GoldChallenge-SQL | contestable oracle protocol | SURVIVE, narrowed |
| 6 | Self-Healing Benchmark | continuous disagreement-driven label versioning | MEDIUM |
| 7 | Contestable RL Reward | probabilistic/adjudicated reward | MEDIUM |
| 8 | **AutonomyOracle-SQL** | minimal sufficient autonomy regime | SURVIVE |
| 9 | Autonomy Calibration Benchmark | forced-regime labels | SURVIVE as benchmark |
| 10 | Over-Autonomy Regret | cost/error from unnecessary agency | SURVIVE as metric |
| 11 | **CausalGuard-SQL** | claim identifiability boundary | SURVIVE |
| 12 | Causal Identifiability Benchmark | descriptive-vs-causal answerability | SURVIVE |
| 13 | CrossDB-Key Agent | dirty-key join reasoning | DOWNGRADE |
| 14 | CrossDB Join Evidence Benchmark | join-evidence sufficiency/safety | SURVIVE as benchmark |
| 15 | Source-Uncertainty Planner | posterior over candidate source DBs | HIGH COLLISION |
| 16 | **Schema-Evolution × Memory Poisoning** | stale verified memory negative transfer | SURVIVE |
| 17 | Memory Invalidation Graph | targeted invalidation after drift | SURVIVE if benchmark shows need |
| 18 | **Model-Upgrade Memory Compatibility** | legacy memory after backbone upgrade | SURVIVE |
| 19 | Text2SQL Memory ABI | compatibility contract | MEDIUM |
| 20 | Agent Upgrade Certification Suite | model+harness+memory upgrade safety | MEDIUM |
| 21 | **HarnessTransfer-SQL** | orchestration-policy generalization | SURVIVE |
| 22 | Strategy Portability Score | quantify harness/policy overfit | SURVIVE as metric |
| 23 | Counterfactual Tool Credit | marginal causal utility of DB calls | MEDIUM |
| 24 | Tool-Call Pruning by Replay | remove causally useless calls | MEDIUM |
| 25 | PolicyCounterfactual-SQL | closest authorized semantic answer | SURVIVE |
| 26 | Minimal Compliant Answer | semantic utility under policy constraints | SURVIVE as variant |
| 27 | Result Contract Language | grain/cardinality/tie/null/time contract | COMPONENT ONLY |
| 28 | AI-Function Necessity Compiler | whether probabilistic operator is required | WATCH |
| 29 | Deterministic AI-Fallback Compiler | replace AI function when safe | WATCH |
| 30 | Stochastic AI-SQL Verifier | statistical correctness of probabilistic SQL | WATCH |
| 31 | AI-SQL Quality Contract | repeatability/confidence requirement | WATCH |
| 32 | Heterogeneous Evidence Join Contract | claims across DB+docs+files | MEDIUM |
| 33 | Multimodal Source-to-Column Evidence | evidence alignment across modalities | MEDIUM |
| 34 | Minimal Required Interaction Benchmark | necessary interaction count/type | MEDIUM |
| 35 | Data-as-Instruction Isolation | prompt-like content in data/schema/docs | HIGH COLLISION with data-agent security |
| 36 | Unsupported-Claim Debt | unsupported claims in data-agent reports | HIGH COLLISION with provenance work |

---

# 4. Final Ranked Shortlist

| Rank | Idea | Provisional novelty | Feasibility | Best paper shape | Main prior-art risk |
|---:|---|---:|---:|---|---|
| 1 | **EvidenceSketch-SQL** | 7.5/10 | 8.5/10 | method/interface | BAP-SQL may expand toward observation representation |
| 2 | **Schema-Evolution × Memory Poisoning** | 7.5/10 | 8/10 | benchmark + safety method | simple versioning may solve most failures |
| 3 | **CausalGuard-SQL** | 7–7.5/10 | 7/10 | data-agent safety | may look like generic causal guardrails |
| 4 | **HarnessTransfer-SQL** | 7/10 | 6.5/10 | empirical/meta-eval | expensive, may be analysis-only |
| 5 | **AutonomyOracle-SQL** | 6.5–7/10 | 7/10 | adaptive inference | AGENTIQL/BAP/general routing prior |
| 6 | **PoU-SQL** | 6.5–7/10 | 8/10 | epistemic benchmark/method | structural abstention + LatentRefusal |
| 7 | **Model-Upgrade Memory Compatibility** | 6.5–7/10 | 8/10 | benchmark/systems | general portable-memory work |
| 8 | **GoldChallenge-SQL** | 6–6.5/10 | 8/10 | evaluation systems | SAR-Agent + GBV-SQL already audit gold errors |
| 9 | PolicyCounterfactual-SQL | 7/10 | 6.5/10 | governance | may reduce to access-control rewriting |
| 10 | CrossDB Join Evidence Benchmark | 5.5–6/10 | 7.5/10 | benchmark | DAB and deterministic production solutions |
| 11 | Stochastic AI-SQL Verifier | 6.5/10 | 7/10 | AI-DB reliability | fast-moving SAGE/Stretto-like work |
| 12 | AI-Function Necessity Compiler | 6/10 | 6.5/10 | AI-DB planning | could be subsumed by future optimizers |

---

# 5. Top Ideas in Detail

## 5.1 EvidenceSketch-SQL — Database Observations as Sufficient Statistics

### Hypothesis

For many Text2SQL decisions, raw tuples are an inefficient sensor output. A task-adaptive database tool can return a structured evidence sketch that resolves uncertainty with fewer tokens and less row exposure.

Example observation request:

```yaml
question_uncertainty: "Is customer→order join one-to-many or many-to-many after promotion bridge?"
requested_sketch:
  - left_key_distinct
  - right_key_distinct
  - matched_key_fraction
  - multiplicity_histogram
  - max_matches_per_key
  - sample_collision_witnesses
```

### Nearest work

BAP-SQL optimizes observation formation under budget; SDE-SQL/PV-SQL/PExA optimize probe actions.

### Defensible delta

Optimize **which representation of an observation** the DB returns (raw rows vs samples vs statistics vs proof witnesses), not only which query to issue.

### Kill experiment

100–300 cases covering joins, duplicates, grain, value ambiguity, temporal coverage, and missingness. Token-match:

- raw top-k rows;
- adaptive samples;
- generic text summary;
- fixed DB profile sketch;
- BAP-style probe returning rows;
- adaptive EvidenceSketch.

**Kill if** adaptive sketches do not improve correctness/uncertainty resolution at matched context, or if gains come only from doing more DB work.

---

## 5.2 Schema-Evolution × Memory Poisoning

### Hypothesis

A memory can be *verified and harmful*: it was correct when written but becomes stale after schema/business evolution, then biases later generation more strongly than no memory.

### Nearest work

EvoSchema studies base-system robustness to controlled schema evolution. AgentSM/Crystallization/GATE/MERIT study reusable Text2SQL memory in largely stable environments.

### Defensible delta

Measure the **negative-transfer surface of previously verified memory under controlled database evolution**.

### Perturbations

- column/table split or merge;
- key uniqueness change;
- SCD policy change;
- bridge-table introduction;
- enum/business-definition change;
- date semantics change;
- metric source deprecation.

### Kill experiment

Establish a memory benefit before change, then apply evolution and compare no-memory, stale-memory, version-tagging, global forgetting, targeted invalidation.

**Kill method novelty if** simple schema-version tagging solves almost everything. Keep as benchmark/analysis only if the poisoning phenomenon is still large.

---

## 5.3 CausalGuard-SQL — Claim-Level Identifiability Boundary

### Hypothesis

A data agent needs an explicit gate between *what SQL observes* and *what the answer claims*. Words like "effect", "impact", "cause", and "what should we do" require stronger evidence than a descriptive aggregation.

```text
question
 ↓
claim type: descriptive / predictive / causal / decision
 ↓
if causal:
  treatment, outcome, estimand
  temporal order
  observed confounders
  intervention/natural-experiment evidence
  identifiability status
 ↓
identified → analysis plan
not identified → causal insufficiency certificate
```

### Nearest work

CORGI includes explanatory/predictive/recommendational business queries. General causal-agent work exists, and non-academic causal-safety guardrail concepts exist.

### Defensible delta

Not general causal discovery: a **database/data-agent safety boundary that prevents valid descriptive SQL from being narrated as unsupported causal evidence**.

### Kill experiment

Use SCM-generated relational databases with matched observational distributions but different causal graphs. Mix descriptive and causal-language questions.

**Kill if** a simple prompt rule ("do not infer causality") performs equally well or objective identifiability labels cannot be constructed.

---

## 5.4 HarnessTransfer-SQL — Does the Agent Policy Generalize?

### Hypothesis

A large fraction of reported "agentic reasoning" gain is policy/harness overfitting. An orchestration policy optimized on BIRD may rank differently on Spider2, interactive SQL, AI-function SQL, or broader data-agent tasks even with the same backbone.

### Evidence motivating the idea

Agentic-SQL Revisited reports uneven transfer across benchmark families. DataSpace reports substantial harness-dependent spread at a fixed backbone. Spider2-AIFunc reports that elaborate conventional Text2SQL agents do not automatically transfer to AI-function tasks.

### Proposed objects

- `Strategy Transfer Matrix[design-domain, eval-domain]`
- `Harness Overfit Gap`
- `Policy Portability Score`

### Kill experiment

Choose 4–6 canonical policies (direct, retrieve+generate, probe+generate, bounded repair, candidate-selection, long-horizon agent), hold backbone/context as fixed as practical, and evaluate across several task families.

**Kill if** policy rankings are stable and cross-domain transfer gaps are small.

---

## 5.5 AutonomyOracle-SQL — Counterfactual Minimal Sufficient Autonomy

### Hypothesis

Each query has a model-specific *cheapest successful autonomy regime* that cannot be predicted well by generic difficulty or token budget alone.

Force the same task through:

```text
R0 direct SQL
R1 internal reasoning / few-shot
R2 semantic plan + deterministic lowering
R3 bounded retrieve/probe/repair
R4 long-horizon external agent
R5 reasoning-internalized trained policy (when available)
```

The oracle label is the cheapest successful regime under a declared utility function. Train a router from forced-regime outcomes.

### Nearest work

Agentic-SQL autonomy taxonomy, AGENTIQL routing, BAP-SQL budget planning, SQL-Trail adaptive turns, and general counterfactual search routing.

### Defensible delta

Counterfactual routing among **qualitatively distinct autonomy regimes**, with explicit under-autonomy and over-autonomy regret.

### Kill experiment

Force 300–1000 tasks through regimes and compare the learned router against question difficulty, risk models, AGENTIQL-style routing, and continuous budget policies.

**Kill if** regime identity adds little information beyond a scalar difficulty/risk/budget score.

---

# 6. Narrowed Backups After Collision Checks

## PoU-SQL

Do **not** claim answerability gating or structural abstention. The only interesting form is a **minimal missing-information certificate**:

```yaml
requested_claim: ...
searched_authorized_sources: ...
missing_semantic_fact: ...
why_available_relations_cannot_determine_it: ...
minimal_addition_that_would_make_it_answerable: ...
```

The key evaluator is a *recovery test*: supply exactly the certified missing fact and verify the question becomes answerable.

## GoldChallenge-SQL

Do **not** claim annotation-error detection. SAR-Agent already does this and GBV-SQL audits Gold Errors. The surviving idea is a **contestable oracle lifecycle**:

```text
model ≠ gold
→ verify both
→ model_wrong / gold_wrong / both_valid / unresolved
→ update versioned confidence
→ propagate to evaluation/RL reward
```

Key metric: human-review saved at fixed correction recall and ranking/reward stability.

## CrossDB Join Evidence Benchmark

Do **not** claim a new generic join agent. Instead test what evidence is necessary to safely accept a dirty cross-source join: overlap, normalization rule, uniqueness, cardinality, temporal consistency, collision witnesses, and false-match stress tests.

---

# 7. Internal Adversarial Review

> Same-session red team only; not an independent research-review receipt.

### EvidenceSketch-SQL

Reviewer attack: "hand-crafted feature engineering saves tokens." Required defense: task-adaptive sketch choice, token/DB-work matching, and ablations separating action selection from observation representation.

### Schema-Evolution × Memory Poisoning

Reviewer attack: "attach a schema version to memory." Required response is empirical: test whether business-semantic changes, constraint changes, and partial dependencies make invalidation non-trivial. If version tags solve it, keep only the benchmark result.

### CausalGuard-SQL

Reviewer attack: "causal inference stapled onto SQL." Required defense: focus exclusively on the mismatch between evidence type and answer claim, not causal-discovery novelty.

### HarnessTransfer-SQL

Reviewer attack: "benchmark survey, not research." Required defense: controlled backbone/policy decomposition and a reproducible transfer matrix that changes how agent methods should be evaluated.

### AutonomyOracle-SQL

Reviewer attack: "AGENTIQL/BAP with more router classes." Required defense: forced counterfactual outcomes must reveal non-monotonic regime effects and over-autonomy failures that scalar difficulty/budget cannot capture.

---

# 8. Five Cheapest Empirical Kill Gates

No experiments were run in this session.

## Gate E — EvidenceSketch

**Continue only if** adaptive evidence representation improves accuracy/uncertainty resolution at matched context and comparable DB work.

## Gate M — Memory Poisoning

**Continue only if** stale verified memory causes material negative transfer under controlled evolution and cannot be almost entirely fixed by trivial version tags.

## Gate C — CausalGuard

**Continue only if** unsupported causal answers drop substantially without excessive false refusal on descriptive questions.

## Gate H — HarnessTransfer

**Continue only if** orchestration-policy rankings or effect sizes change materially across task families at a fixed backbone.

## Gate A — AutonomyOracle

**Continue only if** minimal-autonomy labels contain predictive information beyond scalar difficulty/risk/budget and improve the accuracy-cost Pareto frontier.

---

# 9. Suggested Research Portfolio

### Interface / epistemics
- EvidenceSketch-SQL
- PoU-SQL (narrow certificate version only)

### Persistent-system reliability
- Schema-Evolution × Memory Poisoning
- Model-Upgrade Memory Compatibility

### Meta-evaluation / scientific reliability
- HarnessTransfer-SQL
- GoldChallenge-SQL (contestable oracle only)

### Beyond descriptive SQL
- CausalGuard-SQL
- PolicyCounterfactual-SQL

### Adaptive / AI-native systems
- AutonomyOracle-SQL
- Stochastic AI-SQL Verifier
- AI-Function Necessity Compiler

---

# 10. Key Literature Anchors

Recent/high-priority papers and resources used in this fresh run include:

- Agentic-SQL Revisited — arXiv:2608.15389
- Bounded Semantic Planning and Deterministic Compilation — arXiv:2608.16663
- SemPlan — arXiv:2608.13612
- BAP-SQL — arXiv:2608.02876
- SafeQL — arXiv:2608.09260 / PVLDB 2026
- MDB-Link — arXiv:2608.09588
- Crystallization in Text-to-SQL — arXiv:2608.07213
- SAGE — arXiv:2608.20630
- Never the Number / Structural Abstention — arXiv:2608.13926
- SQL-Trail — ACL 2026
- MTSQL-R1 — ACL 2026
- ReEx-SQL — ACL 2026
- R³-SQL — ACL 2026
- DPC — ACL 2026
- VET — Findings ACL 2026
- PExA — ACL 2026
- PV-SQL — Findings ACL 2026
- SDE-SQL — ACL 2026
- GBV-SQL — ACL 2026
- LatentRefusal — Findings ACL 2026 / arXiv:2601.10398
- AgentSM — arXiv:2601.15709
- MERIT — arXiv:2606.00547
- EnterpriseMem-Bench — arXiv:2605.26394
- GATE — arXiv:2606.05634
- Semantic-layer-mediated Text2SQL agent — arXiv:2606.31041
- SemanticAgent — arXiv:2604.21414
- EvoSchema — arXiv:2603.10697
- EvoSQL — arXiv:2607.20489
- ABISS — arXiv:2607.23340
- Interactive Text-to-SQL via Expected Information Gain — arXiv:2507.06467
- STEPS — EMNLP 2023
- BIRD-INTERACT — ICLR 2026 Oral
- Pervasive Annotation Errors Break Text-to-SQL Benchmarks and Leaderboards — arXiv:2601.08778
- ReViSQL — arXiv:2603.20004
- Spider 2.0-AIFunc — arXiv:2607.06229
- Larch — arXiv:2606.07923
- Stretto — arXiv:2602.04430
- Horrila — arXiv:2604.09944
- Data Agent Benchmark — arXiv:2603.20576
- DataSpace — arXiv:2608.03451
- Data Agents survey — arXiv:2602.04261
- Data Agents Under Attack — arXiv:2606.08661
- CORGI / Agent Bain vs Agent McKinsey — arXiv:2510.07309
- Beyond Text-to-SQL: Governed Enterprise Analytics APIs — arXiv:2605.21027
- TRACER — arXiv:2605.09934
- HEXGEN-TEXT2SQL — arXiv:2505.05286
- AGENTIQL — arXiv:2510.10661

---

# 11. Workflow Status

| Skill phase | Status | Evidence |
|---|---|---|
| research-lit | DONE | fresh 2025–2026 literature landscape above |
| batch idea generation | DONE | 36 raw candidates |
| novelty-check | DONE internally, **provisional** | two-pass collision filtering and closest-work analysis |
| formal novelty reviewer receipt | BLOCKED | independent reviewer backend unavailable here |
| research-review | INTERNAL ONLY | adversarial review above |
| refinement | DONE for batch shortlist | 12 survivors + five kill gates; no single idea frozen |
| pilot experiments | NOT RUN | requires real execution environment |

Because the user asked for a **batch of ideas**, this run intentionally does not collapse everything to a single proposal. The next scientifically useful action is to run the five cheap kill gates, eliminate weak candidates, then invoke the repository's formal independent novelty/research-review stages on the surviving 1–3 ideas.