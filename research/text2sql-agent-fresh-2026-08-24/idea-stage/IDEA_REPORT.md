# Fresh Idea Discovery Report — Text2SQL × Database/Data Agents

**Date:** 2026-08-24  
**Branch:** `research/text2sql-agent-fresh-2026-08-24`  
**Pipeline:** research-lit → idea generation → novelty screening → adversarial review → refinement  
**Independence rule:** this run was started from `main` and does not use conclusions, rankings, or candidate ideas from earlier Text2SQL research branches.  
**Empirical pilots:** NOT RUN in this connector-only session.  
**Formal external reviewer receipt:** UNAVAILABLE in this session; all novelty/review verdicts below are provisional and must be re-checked with the repository's configured cross-model reviewer before implementation.

---

## Executive Summary

The 2026 Text2SQL frontier is no longer dominated by single-shot generation. Recent work has aggressively occupied multi-turn RL, adaptive test-time compute, active database probing, structured memory, semantic planning, candidate selection, execution verification, unknown-database routing, benchmark repair, and AI-native SQL optimization.

That changes the right ideation strategy. The strongest new ideas should not be another planner/critic/verifier loop. They should expose a research variable that current systems either do not represent or do not evaluate directly.

This fresh run generated **36 raw ideas**, aggressively removed ideas whose mechanism is already crowded, and retains **12 serious candidates**. The five strongest paper-shaped bets are:

1. **EvidenceSketch-SQL** — make database observations themselves a learned/optimized interface: sufficient-statistic sketches rather than raw probe rows.
2. **PoU-SQL (Proof of Unanswerability)** — require a checkable certificate of *why* a database question cannot be answered, rather than only abstaining or asking for clarification.
3. **GoldChallenge-SQL** — make benchmark/training labels contestable: disagreements can trigger verification of the gold answer and versioned label-confidence repair.
4. **AutonomyOracle-SQL** — learn the minimum sufficient autonomy regime per query by counterfactually executing the same task through direct, semantic-plan, iterative-agent, and internalized-reasoning regimes.
5. **CausalGuard-SQL** — prevent an analytics agent from turning descriptive SQL evidence into unsupported causal/recommendational claims; require identifiability evidence or produce a causal insufficiency certificate.

Several other directions survive as strong backups: schema-evolution × memory poisoning, cross-database entity-resolution joins, harness/strategy transfer, model-upgrade memory compatibility, policy-aware semantic relaxation, stochastic verification for AI-native SQL, and AI-function necessity planning.

---

# 1. Literature Landscape — What Is Actually Crowded in August 2026

## 1.1 Autonomy and multi-turn interaction are now first-class objects

**Agentic-SQL Revisited** (arXiv:2608.15389, 2026) organizes Text2SQL systems by autonomy level and reports that gains on Spider do not transfer uniformly to BIRD/Spider2; greater autonomy buys robustness but at non-trivial cost. This makes "agent vs no agent" too coarse a research question.

**SQL-Trail** (ACL 2026) and **MTSQL-R1** (ACL 2026) both train long-horizon database interaction. SQL-Trail uses multi-turn RL with interleaved feedback and adaptive turn budget; MTSQL-R1 formulates long-horizon Text2SQL as an MDP with execution and persistent dialogue memory.

**BAP-SQL** (arXiv:2608.02876) makes observation planning explicitly budget-aware. Under tight budgets it reports accuracy gains while using slightly fewer tokens, but gains attenuate as budget/model capacity increase.

**AGENTIQL** (arXiv:2510.10661) already routes between modular agentic and simpler parser pipelines.

### Consequence

Generic claims such as "use an agent only for hard queries," "adaptively choose the number of reasoning turns," or "train a router between simple and agentic pipelines" are already crowded. A new contribution needs a sharper object, such as **counterfactual minimal sufficient autonomy over multiple qualitatively different regimes**.

---

## 1.2 Active observation/probing is crowded, but the observation *representation* is less explored

**SDE-SQL** (ACL 2026) explores a database with SQL probes before generation.

**PV-SQL** (Findings ACL 2026) probes concrete records to resolve ambiguity and then verifies extracted conditions.

**PExA** (ACL 2026) treats exploration like software test coverage and runs atomic SQL tests in parallel.

**BAP-SQL** chooses observation actions under budget.

### Gap

These works mostly optimize **which observations to acquire**. They do not make the *database-to-agent observation channel itself* the main research object. A database tool usually returns rows/text and the LLM must consume them. This motivates **EvidenceSketch-SQL**: query-specific, adaptive sufficient-statistic sketches such as cardinality, key coverage, join coverage, null rates, quantiles, uniqueness evidence, temporal ranges, and compact provenance fingerprints.

---

## 1.3 Candidate generation/selection/verification is extremely crowded

**R³-SQL** groups candidates by execution result, ranks result groups, and resamples when the correct answer is likely absent.

**DPC** (ACL 2026) constructs a Minimal Distinguishing Database and uses dual-paradigm consistency (SQL vs Python/Pandas) for training-free candidate selection.

**VET** (Findings ACL 2026) executes stepwise Python semantics against the real database to make intermediate reasoning observable.

**SafeQL** (PVLDB 2026) refines only erroneous components within a safe search space.

**SCOPE-SQL** explores structured semantic credit for repairing executable-but-wrong SQL.

### Consequence

"Generate multiple SQLs and verify them," "use a judge," "build a counterexample DB," or "repair only the wrong clause" is not sufficient novelty by itself.

---

## 1.4 Memory is now a real Text2SQL subfield

**AgentSM** (arXiv:2601.15709) stores structured programs from execution traces and reuses phase-specific semantic memory, reporting lower token/trajectory cost and improved Spider2-Lite performance.

**MERIT** (arXiv:2606.00547) studies dual-level long-term memory and dynamic retrieval because memory usefulness changes across reasoning stages.

**Crystallization in Text-to-SQL** (arXiv:2608.07213) carefully separates replay, cross-question retention, and held-out same-database transfer. Verified corrected queries improve held-out first-attempt performance, and database-specific content is the main ingredient.

**SDAM** (arXiv:2608.12338) evolves structure-aware semantic memories and handles contradictory rules.

**GATE** (arXiv:2606.05634) uses execution-grounded hypotheses and stores validated semantic grounding memory.

General-agent work also covers portable memory, governed memory, learned memory management, and model-family transfer.

### Consequence

Generic "store successful SQL," "failure memory," "hierarchical memory," "retrieve by phase," or "portable memory" ideas should be treated as baselines. More promising questions involve **compatibility under model upgrades** and **stale-memory failure under schema evolution**.

---

## 1.5 Semantic planning/layers are strong and increasingly deterministic

**Bounded Semantic Planning and Deterministic Compilation** (arXiv:2608.16663) combines stochastic semantic grounding with deterministic graph traversal, grain lowering, SQL compilation, and checks.

**SemPlan** (arXiv:2608.13612) benchmarks direct SQL, bounded agents, semantic-request planning, and clarification/state plans. Structured plans change the failure distribution but do not dominate every policy/cost dimension.

A **Semantic-Layer-Mediated Agent** (arXiv:2606.31041) uses a curated semantic layer and compact Semantic Model Query IR before deterministic dialect compilation.

**SemanticAgent** (arXiv:2604.21414) targets semantic validity in Text2SQL synthesis/diagnosis.

### Consequence

"Add a semantic IR," "compile intent to SQL," "semantic layer + deterministic SQL" is crowded. Result-contract representations can still be useful components, but probably not a standalone paper claim.

---

## 1.6 Ambiguity, clarification, abstention, and interactive correction are established

**ABISS** (arXiv:2607.23340) unifies multiple ambiguous/unanswerable categories with simulated users and identifies clarification-conditioned SQL generation as a bottleneck.

**Interactive Text-to-SQL via Expected Information Gain** (arXiv:2507.06467) selects clarification questions by EIG over competing SQL interpretations.

**PRACTIQ** (NAACL 2025) benchmarks ambiguous and unanswerable questions.

**Reliable Text-to-SQL with Adaptive Abstention** studies abstention/HITL/conformal schema linking.

**STEPS** (EMNLP 2023) already lets users edit step-by-step explanations and updates affected SQL clauses instead of regenerating the whole query.

### Consequence

Generic "ask a good clarification" and "apply a minimal SQL patch after clarification" are not fresh enough. A more distinct target is **a proof/certificate of unanswerability**, with machine-checkable missing information rather than only a classification or natural-language explanation.

---

## 1.7 Benchmark correctness has become a major scientific issue

**Pervasive Annotation Errors Break Text-to-SQL Benchmarks and Leaderboards** (arXiv:2601.08778) reports very high annotation-error rates in expert audits of benchmark subsets and shows corrections can change system ranks.

**Text-to-SQL Benchmarks are Broken** (CIDR 2026) similarly argues benchmark defects distort evaluation.

**ReViSQL** (arXiv:2603.20004) shows verified training data substantially improves the same RL algorithm and reports extensive errors in an inspected BIRD subset.

VLDB 2026 work on **verification algorithms for Text-to-SQL** treats verification itself as a task and observes that a large fraction of apparent model failures may actually be flawed labels.

### Gap

Existing work audits/corrects benchmark labels, but the benchmark typically remains a static oracle during training/evaluation. This opens **GoldChallenge-SQL**: a contestable oracle with disagreement-triggered verification, versioned confidence, and review prioritization.

---

## 1.8 AI-native SQL is creating a new query-semantics problem, not just a new optimizer problem

**Spider 2.0-AIFunc** (arXiv:2607.06229) evaluates native AI functions in SQL across Snowflake databases; frontier performance remains far below saturation and traditional elaborate Text2SQL agents do not necessarily transfer.

**Larch** (arXiv:2606.07923), **Stretto** (arXiv:2602.04430), **Horrila** (arXiv:2604.09944), and **SAGE** (arXiv:2608.20630) optimize AI/semantic query operators, execution strategies, quality/cost tradeoffs, and algebraic abstractions.

### Consequence

"Optimize LLM functions in SQL" is crowded. More open questions may be **whether an AI function is semantically necessary at all** and **how to verify nondeterministic AI-SQL answers statistically**.

---

## 1.9 Text2SQL is becoming only one capability inside broader data agents

**Data Agent Benchmark (DAB)** (arXiv:2603.20576) includes multi-database integration, ill-formatted key joins, unstructured text, and domain knowledge.

**DataSpace** (arXiv:2608.03451) evaluates agents over thousands of heterogeneous artifacts including CSV, JSON, SQLite, Markdown, PDFs, and video; harness choice itself produces large performance differences.

**FDABench** (KDD 2026) and **AgenticDataBench** broaden evaluation to heterogeneous analytical workflows.

The **Data Agents** survey (arXiv:2602.04261) frames autonomy levels beyond one-shot analytics.

**CORGI** targets business questions beyond descriptive SQL, including explanatory, predictive, and recommendational tasks.

### Gap

The next generation of database agents must decide when SQL evidence is *not enough* for the requested claim—especially causal or strategic questions—and must integrate entities across heterogeneous databases safely.

---

# 2. Explicitly Saturated Directions — Do Not Use as Standalone Novelty

| Direction | Why downgraded | Representative nearby work |
|---|---|---|
| Generic active schema exploration | already mature | SDE-SQL, PV-SQL, BAP-SQL, MDB-Link |
| Generic multi-turn SQL RL | directly occupied | SQL-Trail, MTSQL-R1, ReEx-SQL |
| Adaptive number of turns | directly occupied | SQL-Trail, BAP-SQL |
| Planner/critic/verifier loop | heavily occupied | SafeQL, R³-SQL, VET, DPC, many agents |
| Multi-candidate voting/ranking | heavily occupied | R³-SQL, DPC |
| Generic SQL repair | benchmarked/occupied | SafeQL, SWE-SQL/BIRD-CRITIC, SCOPE-SQL |
| Basic semantic memory | occupied | AgentSM, GATE, Crystallization |
| Fancy memory retrieval | occupied | MERIT, SDAM, MCMA/general memory work |
| Semantic IR/compiler | occupied | semantic-layer-mediated agent, bounded semantic planning |
| Generic clarification/EIG | occupied | ABISS, PRACTIQ, EIG interactive SQL |
| Minimal clause patch after user feedback | old close precedent | STEPS |
| Unknown-DB selection | occupied | MDB-Link, TACO |
| Generic claim provenance | general agents already close | TRACER |
| Data-agent security taxonomy | now explicit | Data Agents Under Attack |
| Agentic Text2SQL GPU scheduling | occupied | HEXGEN-TEXT2SQL |
| AI-SQL physical optimization | rapidly crowded | Larch, Stretto, Horrila, SAGE |

---

# 3. Raw Idea Bank — 36 Fresh Candidates

The table deliberately includes risky ideas; later sections eliminate or downgrade the weak ones.

| # | Idea | Core research variable | Initial novelty risk |
|---:|---|---|---|
| 1 | **EvidenceSketch-SQL** | representation of DB observations, not just selection | Low–Medium |
| 2 | Privacy-Aware Evidence Sketches | sufficient evidence under row-exposure/privacy budget | Medium |
| 3 | **PoU-SQL** | machine-checkable proof of unanswerability | Low–Medium |
| 4 | Ambiguity Proof Object | proof of exactly which latent fact clarification resolved | Medium |
| 5 | **GoldChallenge-SQL** | contestable benchmark/training oracle | Low–Medium |
| 6 | Self-Healing Text2SQL Benchmark | continuous disagreement-driven label repair | Medium |
| 7 | Contestable RL Reward | probabilistic/adjudicated reward instead of frozen gold | Medium |
| 8 | **AutonomyOracle-SQL** | minimal sufficient autonomy regime per task | Medium |
| 9 | Autonomy Calibration Benchmark | counterfactual autonomy-tier labels | Medium |
| 10 | Over-Autonomy Regret | cost/error induced by unnecessary agency | Medium |
| 11 | **CausalGuard-SQL** | causal identifiability before answering causal language | Low–Medium |
| 12 | Causal Identifiability Benchmark | distinguish descriptive answerability vs causal identifiability | Low–Medium |
| 13 | CrossDB-Key Agent | verifying entity-resolution joins across databases | Low–Medium |
| 14 | Entity-Link Join Benchmark | malformed/weak keys as a first-class task | Low–Medium |
| 15 | Source-Uncertainty Planner | maintain posterior over candidate source DBs | Medium–High |
| 16 | **Schema-Evolution × Memory Poisoning** | negative transfer from stale verified memories | Low–Medium |
| 17 | Memory Invalidation Graph | provenance-based targeted memory invalidation after schema drift | Medium |
| 18 | **Model-Upgrade Memory Compatibility** | whether memories survive backbone upgrades | Low–Medium |
| 19 | Text2SQL Memory ABI | standardized memory representation + compatibility tests | Medium |
| 20 | Agent Upgrade Certification Suite | certify model/harness/memory upgrade jointly | Medium |
| 21 | **HarnessTransfer-SQL** | generalization of orchestration strategy across benchmarks/domains | Low–Medium |
| 22 | Strategy Portability Score | measure agent-policy overfitting independently of model | Medium |
| 23 | Counterfactual Tool Credit | marginal causal utility of each DB/tool call | Medium |
| 24 | Tool-Call Pruning by Replay | remove calls that do not change final correctness | Medium |
| 25 | PolicyCounterfactual-SQL | closest authorized answer to disallowed request | Low–Medium |
| 26 | Minimal Compliant Answer | optimize semantic utility subject to policy | Low–Medium |
| 27 | Result Contract Language | explicit grain/cardinality/tie/null/time contract | Medium–High |
| 28 | **AI-Function Necessity Compiler** | whether AI operator is needed at all | Medium |
| 29 | Deterministic Fallback Compiler | replace AI operator with deterministic SQL when safe | Medium |
| 30 | **Stochastic AI-SQL Verifier** | statistical correctness/equivalence under nondeterministic operators | Medium |
| 31 | AI-SQL Quality Contract | declarative confidence/repeatability requirement | Medium |
| 32 | Heterogeneous Evidence Join Contract | verify claims joining DB + docs + files | Medium |
| 33 | Multimodal Source-to-Column Mapping | explicit evidence alignment across table/doc modalities | Medium |
| 34 | Minimal Required Interaction Benchmark | label how many/which interactions are necessary | Medium |
| 35 | Data-as-Instruction Isolation | robustness to prompt-like content in data/schema/docs | Medium–High |
| 36 | Unsupported-Claim Debt | quantify claims in data-agent output lacking executable evidence | Medium–High |

---

# 4. Novelty Screening and Eliminations

## 4.1 Ideas killed or strongly downgraded

### Minimal clarification patch

Killed as a standalone direction. STEPS already supports user editing of intermediate explanations and updates affected SQL clauses rather than fully regenerating the query. ABISS also identifies clarification-conditioned SQL generation as an active bottleneck.

### Generic claim provenance / claim graph

Downgraded. TRACER already builds claim-level dependency from tool observations in multimodal agents. A database-specific variant would need materially stronger executable relational provenance to justify a paper.

### Generic architecture router

Killed. AGENTIQL already routes between agentic/modular and simpler Text2SQL paths. BAP-SQL and SQL-Trail already adapt budget/turns. Only **multi-regime counterfactual minimal sufficient autonomy** survives.

### Generic AI-SQL optimizer

Killed. SAGE, Stretto, Horrila, and Larch already occupy physical/quality/cost planning for semantic/AI operators. The survivable angle is upstream: **is an AI function semantically necessary?**

### Generic cross-model portable memory

Killed. General work on portable agent memory and learned memory management already makes this too broad. The narrower Text2SQL question of **legacy-memory compatibility after a model upgrade** survives.

### Generic minimal data access

Downgraded. Minimal-necessary-access auditing is already emerging in high-stakes NL2SQL. PolicyCounterfactual-SQL survives because its object is *semantic relaxation under authorization*, not simply access minimization.

---

# 5. Deep Novelty Review — Top 12 Survivors

## Idea 1 — EvidenceSketch-SQL

### Thesis

A database agent should not always observe raw rows. For many reasoning decisions, the sufficient evidence is a compact structured sketch:

```text
row_count
key_distinct_count
null_fraction
join_match_rate
one_to_many / many_to_many evidence
min/max temporal coverage
quantiles
value-frequency heavy hitters
lineage/provenance fingerprint
small adversarial samples
```

The controller chooses both **what to query** and **what observation representation** to request.

### Closest work

- BAP-SQL: budget-aware observation planning.
- SDE-SQL / PV-SQL: database probes.
- PExA: test-like database exploration.

### Delta

Nearby systems mainly optimize *which DB action to take*. EvidenceSketch makes the **sensor/interface** an explicit learned decision variable. The claim is not "summarize rows"; it is that task-conditioned statistical/evidence sketches give a better correctness–context–privacy frontier than raw rows or generic text summaries.

### Minimal falsification

Construct ambiguous join/filter/grain/value tasks. Match token budget across:

1. raw top-k rows;
2. random/reservoir samples;
3. generic LLM summary of rows;
4. BAP-style selected probe returning rows;
5. fixed schema-statistics sketch;
6. task-adaptive EvidenceSketch.

**Kill** if adaptive sketches do not improve ambiguity-resolution accuracy at matched context or if raw adaptive samples match them.

### Assessment

**Novelty: 7.5/10 provisional. Feasibility: high. Paper shape: strong.**

---

## Idea 2 — PoU-SQL: Proof of Unanswerability

### Thesis

When a question is unanswerable, the agent should emit a **checkable insufficiency certificate**, not merely `ABSTAIN` or a natural-language excuse.

Example certificate:

```yaml
requested_claim: customer lifetime value including offline purchases
missing_fact:
  concept: offline_purchase_amount
  searched_sources: [orders, payments, crm_notes, metric_docs]
  result: no authorized source grounds this concept
minimal_additional_information:
  one of:
    - offline transaction table with customer linkage
    - governed metric definition that supplies offline component
why_SQL_cannot_recover_it:
  - all available monetary relations are online-only
```

### Closest work

- ABISS: ambiguous/unanswerable taxonomy and interaction.
- PRACTIQ: ambiguity/unanswerability benchmark.
- adaptive abstention/HITL systems.

### Delta

These works primarily classify, abstain, or ask. PoU-SQL makes **epistemic insufficiency itself executable/evaluable**. The certificate specifies bounded evidence search and the minimal missing semantic/data fact.

### Minimal falsification

Generate paired tasks where one hidden fact is removed from schema/docs/data. Since construction knows the missing item, certificates can be automatically checked.

Compare:

- binary answerable classifier;
- ABISS-style subcategory classifier;
- natural-language explanation;
- PoU certificate.

Metrics: false-answer rate, missing-fact localization, certificate minimality, recovery after supplying the missing fact.

**Kill** if certificates add little localization/calibration over a strong classifier or require privileged information unavailable to the agent.

### Assessment

**Novelty: 8/10 provisional. Feasibility: high for synthetic/controlled benchmark. Risk: can become an evaluation paper rather than a method paper.**

---

## Idea 3 — GoldChallenge-SQL: Contestable Oracles

### Thesis

The gold SQL/result should not be an unquestionable oracle. When a model and gold disagree, an independent verifier audits **both hypotheses**. Training/evaluation maintain a versioned confidence state rather than a binary frozen label.

```text
model answer ≠ gold
       ↓
construct discriminating executions / semantic checks
       ↓
model-wrong | gold-wrong | both-valid | unresolved
       ↓
update label confidence/version
```

### Closest work

- Pervasive Annotation Errors Break Text-to-SQL Benchmarks.
- CIDR: Text-to-SQL Benchmarks are Broken.
- ReViSQL: verified data substantially improves RLVR.
- VLDB verification work auditing generator errors and flawed gold labels.

### Delta

Existing work demonstrates/audits bad labels and constructs corrected data. GoldChallenge turns **oracle contestability into the online training/evaluation protocol**: disagreement prioritizes verification, unresolved cases have explicit confidence, and leaderboard/reward computation can incorporate versioned adjudication status.

### Minimal falsification

Inject known gold corruptions into a clean benchmark plus a small naturally noisy set. Measure:

- corruption detection precision/recall;
- fraction of human audit saved at fixed correction recall;
- calibration of `gold_wrong/model_wrong/both_valid/unresolved`;
- effect on model ranking or RL reward stability.

**Kill** if verifier cannot separate gold errors from model errors accurately enough to reduce human workload.

### Assessment

**Novelty: 7.5/10 provisional. Feasibility: high. Strong systems/evaluation paper; less clearly a new Text2SQL model.**

---

## Idea 4 — AutonomyOracle-SQL: Minimal Sufficient Autonomy

### Thesis

For the same model and task, run counterfactual execution through several autonomy regimes:

```text
R0 direct generation
R1 in-context/reasoning-only
R2 structured semantic plan + deterministic lowering
R3 bounded retrieve/probe/repair
R4 long-horizon external agent
R5 reasoning-internalized trained policy
```

Define the **minimal successful regime** under an explicit utility function. Train a router from forced-regime outcomes instead of labels like "easy/hard".

### Closest work

- Agentic-SQL Revisited: autonomy taxonomy/empirical analysis.
- AGENTIQL: adaptive router between agentic and baseline parser.
- BAP-SQL: budget-aware planning.
- SQL-Trail: adaptive turn budget.
- General 2026 work on counterfactual supervision for search/no-search routing.

### Delta

The new object is not simple routing or budget. It is **counterfactual multi-regime autonomy calibration**, including over-autonomy regret when a powerful regime is unnecessary or harms correctness/cost.

### Minimal falsification

Force 300–1000 tasks through all regimes with a fixed backbone where possible. Train router on the resulting counterfactual table. Compare against:

- question-length/difficulty heuristic;
- AGENTIQL-style learned router;
- BAP risk score;
- binary search/no-search counterfactual routing adapted to SQL.

Metrics: task success, cost, autonomy regret, Pareto frontier.

**Kill** if a simple risk/difficulty predictor achieves the same frontier or if regime identity adds no value beyond continuous token/tool budget.

### Assessment

**Novelty: 6.5–7/10 provisional. Feasibility: medium. Very timely but strong prior-art pressure.**

---

## Idea 5 — CausalGuard-SQL

### Thesis

Analytics agents often receive language such as "impact," "effect," "what caused," "what should we do," or "would X improve Y?" A relational query can return associations while the final natural-language answer silently becomes causal/recommendational.

CausalGuard inserts a gate:

```text
question
  ↓
descriptive / predictive / causal / decision classification
  ↓
if causal:
  target estimand
  treatment/outcome
  available confounders
  temporal ordering
  intervention/identifiability evidence
  ↓
identified → causal analysis plan
not identified → causal insufficiency certificate
```

### Closest work

- CORGI: enterprise questions beyond descriptive SQL including explanatory/predictive/recommendational tasks.
- CausalSteward: human-in-loop multi-agent causal discovery and identifiability issues.

### Delta

The target is not a general causal-discovery agent. It is a **safety boundary between relational/descriptive evidence and causal claims** inside database/data agents.

### Minimal falsification

Create relational SCM-backed datasets where the same observational tables support correlation but differ in true causal structure. Mix descriptive and causal-language questions.

Compare ordinary data agents vs CausalGuard on:

- unsupported causal-answer rate;
- correct identification/abstention;
- usefulness when the effect is identifiable;
- false refusals on descriptive tasks.

**Kill** if a prompt-only "do not infer causality" baseline performs equally well or if task construction cannot support objective identifiability labels.

### Assessment

**Novelty: 7.5/10 provisional. Feasibility: medium. High conceptual value; may publish better as data-agent safety than Text2SQL.**

---

## Idea 6 — Schema-Evolution × Memory Poisoning

### Thesis

A verified Text2SQL memory is correct only relative to a schema/business state. After schema evolution, yesterday's high-confidence memory can become a high-confidence source of error.

Benchmark changes such as:

- table split/merge;
- key replacement;
- renamed/deprecated metric;
- SCD policy change;
- changed enum semantics;
- bridge-table introduction;
- column becomes non-unique;
- timestamp semantic change.

Then measure whether memory helps or harms.

### Closest work

- EvoSchema: systematic schema-evolution perturbations for Text2SQL robustness.
- AgentSM/GATE/Crystallization/MERIT: Text2SQL memory reuse.
- General governed-memory systems: revision/forgetting.

### Delta

The combination—**verified SQL memory under controlled schema evolution and its negative transfer curve**—is not the core task of either literature.

### Minimal falsification

Take memory-enabled Text2SQL systems, establish beneficial pre-change memory, apply EvoSchema-like evolution, and measure:

- stale-memory error rate;
- detection latency;
- recovery after invalidation;
- targeted vs global forgetting.

**Kill** as a method paper if simple schema-version tagging solves nearly all failures. It can still be a benchmark/analysis paper if the phenomenon is substantial.

### Assessment

**Novelty: 7.5/10 provisional. Feasibility: high. Strong benchmark + memory-safety paper.**

---

## Idea 7 — CrossDB-Key Agent

### Thesis

In heterogeneous data-agent tasks the hard part is often not selecting the right database but deciding whether two imperfect identifiers actually represent the same entity and whether a cross-source join is safe.

The agent treats a cross-database join as a hypothesis with evidence:

```text
candidate key mapping
coverage
uniqueness
many-to-many risk
normalization transform
fuzzy/entity match confidence
temporal consistency
collision analysis
```

### Closest work

- MDB-Link: target-database localization/schema narrowing.
- DAB/DataSpace: heterogeneous multi-source tasks, including ill-formatted key joins.

### Delta

Focuses on **cross-source entity-resolution join validity**, not database selection or conventional schema linking.

### Minimal falsification

Create/curate tasks with dirty IDs, aliases, formatting changes, composite keys, temporal remapping, and tempting false joins. Compare LLM join choice vs explicit join-evidence planner.

**Kill** if conventional entity-resolution tooling plus simple thresholds already solves the benchmark or if LLM reasoning adds no benefit.

### Assessment

**Novelty: 7/10 provisional. Feasibility: medium-high. Strong connection to broader data-agent benchmarks.**

---

## Idea 8 — HarnessTransfer-SQL

### Thesis

Many "agent gains" may be benchmark/harness-specific policies. Measure whether orchestration strategies transfer across BIRD, Spider2, AIFunc, interactive DB tasks, and heterogeneous data-agent tasks while holding backbone as fixed as possible.

### Closest work

- Agentic-SQL Revisited: uneven transfer of gains across benchmarks.
- DataSpace: fixed backbone can vary substantially with harness.
- Spider2-AIFunc: elaborate traditional Text2SQL agents do not automatically transfer.

### Delta

Make **strategy generalization** the object rather than final benchmark accuracy.

Define:

```text
Strategy Transfer Matrix[train/design domain → eval domain]
Policy Portability Score
Harness Overfit Gap
```

### Minimal falsification

Implement 4–6 canonical strategies and test them with one or two backbones across multiple task families. If rankings are stable and no material harness overfitting appears, the thesis weakens.

### Assessment

**Novelty: 7/10 provisional. Feasibility: medium; evaluation expensive but scientifically clean.**

---

## Idea 9 — Model-Upgrade Memory Compatibility

### Thesis

Organizations will upgrade the backbone while retaining years of Text2SQL memories/examples. A memory that helps model A may distract or miscalibrate model B.

### Closest work

- Portable Agent Memory: general cross-model memory transfer.
- MCMA/general learned memory management.
- EnterpriseMem-Bench: large model/dataset-dependent memory effects and even model-generation regressions.

### Delta

The target is **legacy semantic/query memory as a compatibility surface during Text2SQL backbone migration**, not generic memory portability.

### Minimal falsification

Build memories using model A, freeze them, swap to models B/C, and measure helpfulness, negative transfer, ranking instability, and benefit of revalidation/rewriting.

**Kill** if raw verified memories transfer uniformly or a generic memory normalizer fixes all model-specific effects.

### Assessment

**Novelty: 6.5–7/10 provisional. Feasibility: high if model access exists. Practical value high.**

---

## Idea 10 — PolicyCounterfactual-SQL

### Thesis

When the exact requested answer is disallowed, the agent should find the **closest policy-compliant semantic answer** instead of only refusing.

Example:

```text
requested: individual salary by employee
policy forbids row-level disclosure
possible relaxation: department-level salary distribution with k-anonymity threshold
```

Optimize semantic utility under authorization/privacy constraints.

### Closest work

- governed enterprise analytics API agents;
- row/column-level security and policy-aware orchestration;
- minimal necessary access work.

### Delta

The main object is **semantic counterfactual relaxation**: how little does the user's analytical intent need to change to become authorized?

### Minimal falsification

Build requests with a lattice of allowed aggregations/generalizations. Compare refuse-only, rule-based relaxation, and LLM semantic relaxation. Measure policy violations and semantic utility.

**Kill** if this reduces to a conventional query-rewriting/access-control lattice with no useful LLM role.

### Assessment

**Novelty: 7/10 provisional. Feasibility: medium. Strong security/governance intersection.**

---

## Idea 11 — Stochastic AI-SQL Verifier

### Thesis

Queries containing LLM/AI functions are probabilistic programs. Equality after one execution is not a reliable correctness criterion. Verification should reason over repeated outcomes, confidence intervals, metamorphic invariants, and acceptable quality risk.

### Closest work

- Spider2-AIFunc uses repeated execution to construct/validate a benchmark.
- Stretto/SAGE model quality/cost tradeoffs and semantic execution.

### Delta

Make **statistical query correctness/equivalence** the central object for AI-native SQL rather than physical optimization.

### Minimal falsification

Construct AI-SQL queries with deterministic relational shell + stochastic operators. Compare one-run execution equivalence, repeated-majority, confidence-bound verification, and metamorphic checks.

**Kill** if existing SAGE/Stretto guarantees already cover the proposed semantics or if repeated execution alone is sufficient.

### Assessment

**Novelty: 6.5/10 provisional. Feasibility: medium-high; high risk of collision with fast-moving AI-DB work.**

---

## Idea 12 — AI-Function Necessity Compiler

### Thesis

Before optimizing an AI function, ask whether an LLM/AI operator is needed at all. Many apparently semantic tasks may be solvable by deterministic SQL, dictionary lookup, regex, embedding index, a cached/proxy model, or an AI function.

### Closest work

- Spider2-AIFunc assumes AI-function tasks.
- SAGE/Stretto/Horrila/Larch optimize AI operators once admitted into the plan.
- production systems already use proxy models/caching strategies.

### Delta

The decision is **semantic necessity of probabilistic computation**, upstream of physical AI-query planning.

### Minimal falsification

Create paired tasks where deterministic and AI solutions coexist. Evaluate planner selection under correctness/cost/latency/privacy constraints.

**Kill** if existing cost-based optimizers naturally subsume the choice or if task construction is artificial.

### Assessment

**Novelty: 6/10 provisional. Feasibility: medium. Keep as backup, not first bet.**

---

# 6. Ranked Shortlist

| Rank | Idea | Novelty | Feasibility | Clean falsification | Main risk | Status |
|---:|---|---:|---:|---:|---|---|
| 1 | **EvidenceSketch-SQL** | 7.5 | 8.5 | 9 | could be "just feature engineering" | RECOMMENDED |
| 2 | **PoU-SQL** | 8.0 | 8.0 | 9 | may become benchmark-only | RECOMMENDED |
| 3 | **GoldChallenge-SQL** | 7.5 | 8.5 | 9 | evaluation systems rather than model | RECOMMENDED |
| 4 | **CausalGuard-SQL** | 7.5 | 7.0 | 8 | drifts beyond Text2SQL | RECOMMENDED |
| 5 | **AutonomyOracle-SQL** | 6.5–7 | 7.0 | 8 | strong routing prior art | RECOMMENDED WITH CAUTION |
| 6 | Schema-Evolution × Memory Poisoning | 7.5 | 8.0 | 9 | simple versioning may solve | STRONG BACKUP |
| 7 | CrossDB-Key Agent | 7.0 | 7.5 | 8 | classical entity resolution may dominate | STRONG BACKUP |
| 8 | HarnessTransfer-SQL | 7.0 | 6.5 | 8 | expensive evaluation | BACKUP |
| 9 | PolicyCounterfactual-SQL | 7.0 | 6.5 | 8 | may reduce to access-control rewriting | BACKUP |
| 10 | Model-Upgrade Memory Compatibility | 6.5–7 | 8.0 | 8 | general memory literature close | BACKUP |
| 11 | Stochastic AI-SQL Verifier | 6.5 | 7.0 | 7 | AI-DB field moves very fast | WATCH |
| 12 | AI-Function Necessity Compiler | 6.0 | 6.5 | 7 | optimizer prior art | WATCH |

---

# 7. Internal Adversarial Review

> **Important:** this section is same-session internal red-team reasoning, not the repository skill's required independent reviewer receipt.

## EvidenceSketch-SQL

A skeptical reviewer will say: "You invented a hand-crafted summary format and unsurprisingly saved tokens." The paper only becomes interesting if the sketches are **task-adaptive**, if they dominate raw/sampled observations under matched token budget, and if the learned choice of evidence type matters. Strong ablations must separate (a) probe selection, (b) representation, and (c) compression.

## PoU-SQL

A skeptical reviewer will say: "Unanswerability classification with extra YAML." The certificate must have a checker with real semantics: bounded evidence search, localization of the absent fact, and a recovery test where supplying exactly that fact makes the task answerable. The strongest claim may be a new epistemic evaluation task rather than a generation architecture.

## GoldChallenge-SQL

A skeptical reviewer will say: "This is dataset cleaning with an LLM judge." The novelty requires a protocol where oracle uncertainty is maintained online, disagreements are prioritized by expected impact, and system ranking/reward changes are demonstrably more stable. Human audit savings are a key quantitative metric.

## CausalGuard-SQL

A skeptical reviewer will say: "This is a causal inference paper glued to an SQL agent." The paper needs a sharp boundary: the contribution is a **claim-type/identifiability gate for data agents**, not causal discovery. It must show ordinary descriptive SQL remains useful while unsupported causal language is reduced.

## AutonomyOracle-SQL

A skeptical reviewer will cite AGENTIQL, BAP-SQL, Agentic-SQL Revisited, SQL-Trail, and general counterfactual search routing. The idea survives only if multiple autonomy regimes are qualitatively different and the counterfactual oracle yields a calibration variable not captured by difficulty/risk/token budget.

## Schema-Evolution × Memory Poisoning

A skeptical reviewer may accept the benchmark phenomenon but reject a complex mitigation if schema-version tags solve it. Therefore the first study should be diagnostic: measure which evolution types invalidate which memory types and whether negative transfer is local or contagious.

---

# 8. Minimal Pre-Implementation Experiments / Kill Gates

No experiments were run in this session. These are the cheapest tests to perform once an execution environment is available.

## Gate E — EvidenceSketch-SQL

- 100–300 hard ambiguity cases spanning join cardinality, value ambiguity, missingness, temporal ranges, duplicate/grain errors.
- Compare raw rows, token-matched samples, generic summaries, fixed sketches, adaptive sketches.
- **Continue only if** adaptive sketches improve ambiguity resolution or correctness at the same context budget and the gain is not explained by more DB work.

## Gate P — PoU-SQL

- Build 200 paired answerable/unanswerable tasks by removing one known required fact/source/definition.
- Evaluate missing-fact localization and recovery after restoring it.
- **Continue only if** certificate localization materially exceeds classification/explanation baselines and false "provably unanswerable" rate is very low.

## Gate G — GoldChallenge-SQL

- Start with a verified subset; inject realistic label corruptions and add a small real noisy set.
- Measure disagreement triage precision/recall and human-review reduction.
- **Continue only if** the protocol catches corrupted gold reliably without falsely exonerating bad model outputs.

## Gate A — AutonomyOracle-SQL

- Force 300 tasks through 4–5 regimes with the same backbone when possible.
- Measure how often the cheapest successful regime differs from simple difficulty ordering.
- **Continue only if** the counterfactual regime label contains information beyond risk score/token budget and a router improves the cost–accuracy frontier.

## Gate C — CausalGuard-SQL

- Create SCM-backed relational tasks with matched observational distributions but different causal structures.
- **Continue only if** the gate reduces unsupported causal answers substantially while preserving descriptive utility.

---

# 9. Recommended Research Portfolio

Rather than betting on one large framework immediately, the most rational portfolio is:

### Track A — Interface/Epistemics

- EvidenceSketch-SQL
- PoU-SQL

These are cheap to falsify and change the DB-agent interface itself rather than adding another reasoning loop.

### Track B — Evaluation Reliability

- GoldChallenge-SQL
- Schema-Evolution × Memory Poisoning
- HarnessTransfer-SQL

These exploit strong 2026 evidence that benchmark labels, memories, and harnesses can dominate apparent model quality.

### Track C — Beyond Descriptive SQL

- CausalGuard-SQL
- CrossDB-Key Agent
- PolicyCounterfactual-SQL

These move toward the broader data-agent setting where relational generation is only one subproblem.

### Track D — Adaptive Systems / AI-Native SQL

- AutonomyOracle-SQL
- Stochastic AI-SQL Verifier
- AI-Function Necessity Compiler

These are timely but have the highest concurrent-work risk and therefore require another novelty check immediately before implementation.

---

# 10. Sources / Literature Anchors

Recent/high-priority anchors used in this fresh run include:

- Agentic-SQL Revisited: arXiv:2608.15389
- Bounded Semantic Planning and Deterministic Compilation: arXiv:2608.16663
- SemPlan: arXiv:2608.13612
- BAP-SQL: arXiv:2608.02876
- SafeQL: arXiv:2608.09260 / PVLDB 2026
- MDB-Link: arXiv:2608.09588
- Crystallization in Text-to-SQL: arXiv:2608.07213
- SDAM: arXiv:2608.12338
- SAGE: arXiv:2608.20630
- SQL-Trail: ACL 2026
- MTSQL-R1: ACL 2026
- ReEx-SQL: ACL 2026
- R³-SQL: ACL 2026
- DPC: ACL 2026
- VET: Findings ACL 2026
- PExA: ACL 2026
- PV-SQL: Findings ACL 2026
- SDE-SQL: ACL 2026
- AgentSM: arXiv:2601.15709
- MERIT: arXiv:2606.00547
- EnterpriseMem-Bench: arXiv:2605.26394
- GATE: arXiv:2606.05634
- Semantic-layer-mediated agent: arXiv:2606.31041
- SemanticAgent: arXiv:2604.21414
- EvoSchema: arXiv:2603.10697
- ABISS: arXiv:2607.23340
- Interactive Text-to-SQL via Expected Information Gain: arXiv:2507.06467
- STEPS: EMNLP 2023
- BIRD-INTERACT: ICLR 2026 Oral
- Pervasive Annotation Errors Break Text-to-SQL Benchmarks: arXiv:2601.08778
- ReViSQL: arXiv:2603.20004
- Spider 2.0-AIFunc: arXiv:2607.06229
- Larch: arXiv:2606.07923
- Stretto: arXiv:2602.04430
- Horrila: arXiv:2604.09944
- Data Agent Benchmark: arXiv:2603.20576
- DataSpace: arXiv:2608.03451
- Data Agents survey: arXiv:2602.04261
- Data Agents Under Attack: arXiv:2606.08661
- Beyond Text-to-SQL: Governed Enterprise Analytics APIs: arXiv:2605.21027
- TRACER: arXiv:2605.09934
- CausalSteward: arXiv:2607.01936
- HEXGEN-TEXT2SQL: arXiv:2505.05286
- AGENTIQL: arXiv:2510.10661

---

# 11. Workflow Status

| Skill phase | Status | Evidence |
|---|---|---|
| research-lit | DONE | literature landscape in this report |
| idea generation | DONE | 36-idea raw bank |
| novelty screening | DONE, provisional | saturated list + top-12 nearest-work analysis |
| formal novelty reviewer | BLOCKED | independent reviewer backend unavailable in this connector session |
| research-review | INTERNAL ONLY | adversarial review section; no external receipt |
| research-refine-pipeline | PARTIAL BY DESIGN | batch shortlist + kill gates; no single idea frozen because user requested bulk ideas |
| pilot experiments | NOT RUN | requires an execution environment |

The correct next step is **not** to implement all twelve ideas. Once an execution environment is available, run the five minimal gates in Section 8 in parallel or sequentially, kill weak directions, then run the repository's formal novelty/research-review stages on the surviving 1–3 ideas.