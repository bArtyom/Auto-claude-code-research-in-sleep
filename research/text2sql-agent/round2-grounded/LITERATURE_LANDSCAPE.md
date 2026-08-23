# Round 2 — Grounded Literature Landscape for Text2SQL × Agents

> **Date:** 2026-08-23  
> **Role in ARIS-style workflow:** `research-lit`  
> **Scope:** Text2SQL / SQL agents, interactive database agents, plus mature mechanisms from database theory, program synthesis, weak supervision, temporal databases, control, and programming languages.  
> **Important:** This is a targeted literature map for idea discovery, not a claim of exhaustive prior-art coverage.

## 0. Why this round is different

The previous brainstorming rounds deliberately maximized breadth. This round reverses the order:

1. map the **current Text2SQL frontier**;
2. identify which attractive ideas are already crowded;
3. import mechanisms only where they create a research variable not already covered;
4. require every proposed direction to admit a cheap falsification test.

The central observation from the literature is that several ideas that looked novel a year ago are now becoming standard: active schema exploration, agent memory, clarification, semantic intermediate representations, multi-candidate scaling, and verifier-style agent loops all have close recent work. Therefore the next contribution should not be “one more agent loop.” It should change either the **supervision channel**, the **persistent state**, the **learning timescale**, or the **database interface itself**.

---

# 1. Current Text2SQL frontier

## 1.1 Interaction is now a benchmarked object

**BIRD-INTERACT** (Huo et al., 2025, arXiv:2510.05318) evaluates both conversational and open-ended agentic interaction with database environments, knowledge bases, metadata, and user simulation. The reported success rates remain low even for frontier models, which supports treating tool-use policy and interaction trajectory as first-class research variables rather than evaluation noise.

**PRACTIQ** (Dong et al., NAACL 2025; arXiv:2410.11076) explicitly introduces ambiguous and unanswerable questions and evaluates clarification behavior.

**AmbiSQL** (Ding et al., 2025/2026, arXiv:2508.15276) detects fine-grained ambiguity and guides users through clarification choices.

**Reliable Text-to-SQL with Adaptive Abstention** (Chen et al., 2025, arXiv:2501.10858) combines abstention, human-in-the-loop interaction, and conformal techniques for schema linking.

### Consequence for novelty

A proposal whose core contribution is simply “detect ambiguity and ask a clarification question” is now crowded. New clarification work needs a substantially different signal or interaction modality—for example, **result-table corrections**, **minimal conflict cores**, or **historical temporal semantics**.

---

## 1.2 Active schema discovery is no longer an empty space

**AutoLink** (Wang et al., 2025, arXiv:2511.17190) reformulates large-schema linking as autonomous iterative exploration and expansion, with strong schema-linking recall even for thousands of columns.

**TRUST-SQL** (Jian et al., 2026, arXiv:2603.16448) goes further: the full schema is not preloaded, the problem is formulated as a POMDP, and a tool-using agent explores and verifies metadata before SQL generation.

**CHESS** (Pourreza & Rafiei et al., 2024, arXiv:2405.16755) already demonstrated the value of retrieval, schema selection, generation, execution, and revision in a specialist pipeline.

### Consequence for novelty

“Active schema exploration,” “POMDP Text2SQL,” or “progressively reveal schema” should be considered **near-prior-art**, unless the objective changes. A potentially open variation is **cross-task dual control**: take an action that helps the current query *and* deliberately improves the warehouse model for future queries.

---

## 1.3 Memory has entered agentic Text2SQL

**AgentSM: Semantic Memory for Agentic Text-to-SQL** (Biswal et al., 2026, arXiv:2601.15709) stores prior execution traces as interpretable structured programs and reuses them to shorten trajectories and token usage on Spider 2.0.

### Consequence for novelty

“Store previous mistakes/trajectories and retrieve similar ones” is not enough. A new memory paper should operate on a different mechanism, e.g.:

- **automatic abstraction invention** rather than trace retrieval;
- **valid-time / transaction-time semantics** for changing business definitions;
- **dependency-directed invalidation** when a belief changes;
- explicit measurement of **negative transfer / memory poisoning**.

---

## 1.4 Semantic IR / semantic layers are now strong baselines

**A Semantic-Layer-Mediated Agent for Natural Language to SQL over Heterogeneous Enterprise Databases** (Kim et al., 2026, arXiv:2606.31041) decouples intent from physical SQL through a curated semantic layer and a compact Semantic Model Query (SMQ) IR, then deterministically compiles to multiple SQL dialects.

**SemanticAgent** (Gao et al., 2026, arXiv:2604.21414) explicitly targets semantic validity during Text2SQL data synthesis with semantic analysis, structured knowledge, synthesis, and diagnosis.

### Consequence for novelty

A generic “semantic IR + compiler” or “semantic verifier” idea is no longer sufficiently distinctive. The unexplored axis is how such semantic assets are **learned, versioned, repaired, or materialized automatically over a workload**.

---

## 1.5 More agent steps are not automatically better

**Agentar-Scale-SQL** (Wang et al., 2025, arXiv:2509.24403) shows that orchestrated internal, sequential, and parallel test-time scaling can substantially improve Text2SQL.

But **Rethinking Agentic Workflows** (Guo et al., 2025, arXiv:2510.10885) reports that extra workflow steps have mixed benefit and that model choice plus simple strategies such as divide-and-conquer and few-shot prompting can dominate over gratuitous orchestration.

More strongly, **ReViSQL** (Zhu et al., 2026, arXiv:2603.20004) argues that cleaner data plus RL with verifiable rewards can close much of the gap without piling on architectural complexity; the authors report substantial annotation errors in the BIRD training subset they inspected.

### Consequence for novelty

Every new agent architecture should be compared against a **strong simple baseline** and a matched-compute baseline. The scientific burden is not “our loop works,” but “the imported mechanism adds something that cannot be recovered by a better base model, more samples, or cleaner data.”

---

## 1.6 SQL debugging itself is now benchmarked

**SWE-SQL / BIRD-CRITIC** (Li et al., 2025, arXiv:2506.18951) studies realistic SQL issue debugging across dialects, rather than only natural-language-to-SQL generation.

### Consequence for novelty

A generic “SQL repair agent” is crowded. A promising gap is to change the feedback interface from a textual bug report to **denotational feedback**: “this output row should not be here,” “this row is missing,” or “this cell is wrong.” That connects directly to database provenance and causality.

---

# 2. Foundational mechanisms that expose genuinely different variables

## 2.1 DreamCoder: learn a library, not a memory cache

**DreamCoder** (Ellis et al., PLDI 2021, DOI:10.1145/3453483.3454080) alternates wake and sleep phases. It jointly learns a reusable program library and a search policy; an e-graph-based refactoring procedure discovers repeated sub-programs and promotes them into abstractions.

The key transfer is not “sleep/replay.” It is **library learning / abstraction invention**.

For a repeated enterprise SQL workload, a system could discover abstractions such as:

- `recognized_revenue()`
- `active_customer(as_of)`
- `latest_valid_snapshot(entity)`
- `deduplicated_orders(grain)`
- `fiscal_period(date)`

These are stronger than retrieved examples: they become new operators in the agent's program language.

**Novelty opening:** current Text2SQL memory work retrieves structured traces; current semantic-layer work assumes a curated semantic layer. An agent that **automatically induces the semantic library that later queries compile against** sits between those two literatures.

---

## 2.2 Database provenance and query causality: supervise with the output, not the prompt

**Why and Where: A Characterization of Data Provenance** (Buneman, Khanna, Tan, ICDT 2001, DOI:10.1007/3-540-44503-X_20) distinguishes where output data came from and why it exists.

**Provenance Semirings** (Green, Karvounarakis, Tannen, PODS 2007) gives an algebraic foundation for propagating provenance annotations through relational queries.

**Causality in Databases / Responsibility for Query Answers and Non-Answers** (Meliou et al., PVLDB 2010) refines lineage into causes and degrees of responsibility for observed answers or non-answers.

These ideas suggest a completely different human feedback channel:

> User does not explain the SQL bug. User edits or labels the **result**. The system asks which input tuples / query operators were responsible for the unwanted or missing output and repairs the query minimally.

This is closer to **reverse data management** than ordinary Text2SQL clarification.

---

## 2.3 Relational lenses: bidirectionality as an interaction primitive

**Incremental Relational Lenses** (Horn, Perera, Cheney, 2018, arXiv:1807.01948) studies bidirectional transformations where edits to a view are translated back into source changes.

We do **not** want to update source data in Text2SQL. But the interaction law is useful:

- forward: question / semantic program → SQL → result;
- backward: desired result correction → minimal semantic/query correction.

This yields a “Result Lens” interaction mode distinct from chat clarification.

---

## 2.4 Bitemporal databases: distinguish when a fact was true from when the agent learned it

Temporal database work distinguishes:

- **valid time:** when a fact holds in the modeled world;
- **transaction time:** when that fact is stored/known by the database.

A memory item such as `revenue := net_recognized_amount` therefore needs two clocks:

- when this definition is valid in the business;
- when the agent learned / revised the definition.

This matters for questions like “what did ARR mean in Q2 2024?” after the organization changed the metric definition in 2025.

**Novelty opening:** Text2SQL semantic memory exists, but temporal validity and historical belief reconstruction are not the core mechanism of AgentSM.

---

## 2.5 Bayesian online change-point detection: detect semantic regime shifts

**Bayesian Online Changepoint Detection** (Adams & MacKay, 2007, arXiv:0710.3742) maintains a posterior over the time since the latest change point.

For a long-lived database agent, observations that may indicate a semantic change include:

- a dbt definition changes;
- a previously stable join cardinality shifts;
- a metric's historical accepted SQL changes;
- user corrections spike around one concept;
- a schema alias now points to a different source.

This creates a falsifiable task: detect semantic regime changes quickly while avoiding unnecessary invalidation of stable memory.

---

## 2.6 Truth Maintenance Systems: record reasons and invalidate dependents

**Doyle's Truth Maintenance System** (Artificial Intelligence, 1979) records reasons for beliefs and uses dependency-directed backtracking when assumptions are contradicted.

**de Kleer's Assumption-Based TMS** (Artificial Intelligence, 1986) maintains sets of assumptions and can reason across mutually inconsistent contexts.

This makes memory repair operational rather than rhetorical. If a warehouse belief changes, the system can invalidate only derived beliefs and cached query templates that depend on it.

This is stronger than appending “new memory” next to stale memory.

---

## 2.7 Weak supervision: learn a SQL judge from many noisy tests

**Snorkel** (Ratner et al., 2017, arXiv:1711.10160) combines noisy, potentially correlated labeling functions without requiring gold labels for every example.

A SQL verifier has exactly this structure. Candidate signals include:

- execution succeeds;
- dbt tests pass;
- PK/FK cardinality is plausible;
- query grain matches requested entity;
- lineage reaches an authoritative model;
- a historical analyst query agrees;
- result invariants hold;
- a separate LLM judge agrees.

None is a perfect oracle. Instead of hard-coding all signals, learn their reliabilities and dependencies from a small calibration set.

---

## 2.8 Materialized views and incremental view maintenance: change the environment

Database systems have decades of work on **materialized view selection** and **incremental view maintenance**. Gupta, Mumick & Subrahmanian (SIGMOD 1993) show how to incrementally maintain views after base-data changes; modern systems such as **Enzyme** (Yadav et al., 2026, arXiv:2603.27775) continue to invest in industrial IVM.

The Text2SQL transfer is structural:

> A long-lived agent should sometimes stop re-solving a repeated semantic pattern and instead create a verified reusable semantic view / metric / macro.

The objective becomes cumulative:

`future reasoning saved + query cost saved - maintenance cost - semantic risk`.

That is not memory retrieval; it is **self-modification of the data interface**.

---

## 2.9 Equality saturation: collapse surface diversity before voting

**egg: Fast and Extensible Equality Saturation** (Willsey et al., POPL 2021, arXiv:2004.03082) uses e-graphs to represent many equivalent expressions compactly.

Multi-candidate Text2SQL systems often spend compute on candidates that are syntactically different but semantically equivalent under safe rewrite rules. A SQL e-graph could first merge candidates in the same conservative equivalence class, then spend expensive verification only on **true semantic disagreement**.

This attacks ensemble redundancy rather than generating even more candidates.

---

## 2.10 Information Bottleneck: compact warehouse state, not per-query retrieval

**The Information Bottleneck Method** (Tishby, Pereira, Bialek, 1999/2000, arXiv:physics/0004057) seeks a compressed representation of X that preserves information relevant to Y.

A database agent could learn a compact persistent warehouse state `Z` that preserves information necessary for a *distribution of future queries*, rather than retrieving top-k schema items independently for every query.

This is only interesting if `Z` is evaluated across a task stream and against strong retrievers; otherwise it degenerates into another schema-linking model.

---

## 2.11 Dual control: current-task action + future model learning

Dual control treats an action as serving two objectives simultaneously: controlling the current system and probing it to reduce uncertainty about future behavior.

The database analogue is not merely active retrieval. A probe chosen during query `t` should be rewarded if it both solves `t` and improves the agent's warehouse model enough to reduce regret on `t+1...T`.

This creates a distinct long-horizon objective from TRUST-SQL, whose unknown-schema exploration is centered on the current query.

---

# 3. Saturated directions: ideas we should explicitly downgrade

The novelty check in the next stage should start by **penalizing** candidates whose main mechanism is already close to the following work:

| Candidate mechanism | Nearby literature | Current status |
|---|---|---|
| Active schema POMDP | TRUST-SQL, AutoLink | crowded |
| Basic semantic memory / trace reuse | AgentSM | crowded |
| Semantic IR + deterministic compiler | semantic-layer-mediated agent | crowded |
| Generic semantic verifier | SemanticAgent + many repair loops | crowded |
| Clarification / ambiguity detection | PRACTIQ, AmbiSQL, adaptive abstention | crowded |
| More candidates + tournament / test-time scaling | Agentar-Scale-SQL, XiYan-style ensembles, DivSkill-SQL | crowded |
| Generic agent review / refine loops | CHESS, agentic workflow literature | crowded |
| Generic SQL debugging | BIRD-CRITIC / SWE-SQL | crowded |
| “More agent steps” as contribution | counter-evidence from Rethinking Agentic Workflows and ReViSQL | weak thesis |

This does **not** mean these mechanisms are useless. It means they should be baselines or components, not the paper's novelty claim.

---

# 4. Open research seams suggested by the literature

The literature map leaves several seams that look less saturated:

1. **Denotational feedback** — repair SQL from corrected output rows/cells using provenance/causality.
2. **Automatic abstraction invention** — induce reusable semantic operators from repeated tasks, instead of retrieving traces or relying on manually curated semantic layers.
3. **Temporal semantics of memory** — version business definitions by valid time and knowledge time; detect regime shifts.
4. **Dependency-aware memory repair** — invalidate derived memories, templates, and past conclusions when a foundational semantic fact changes.
5. **Self-materializing semantics** — create reusable views/metrics/macros when future amortized utility justifies maintenance cost.
6. **Weakly supervised semantic verification** — combine many imperfect enterprise signals rather than assume one LLM judge or one gold SQL.
7. **Cross-task dual control** — spend a probe now partly to improve future queries.
8. **Equivalence-class reconciliation** — use sound rewrites/e-graphs to separate surface diversity from real semantic diversity.
9. **Minimal sufficient warehouse state** — persistent compressed world state evaluated over a task stream, not top-k retrieval per query.

These seams define the candidate-generation space for the next stage.

---

# 5. References

## Text2SQL / agent frontier

- Huo et al. **BIRD-INTERACT: Re-imagining Text-to-SQL Evaluation for Large Language Models via Lens of Dynamic Interactions.** arXiv:2510.05318, 2025.
- Wang et al. **AutoLink: Autonomous Schema Exploration and Expansion for Scalable Schema Linking in Text-to-SQL at Scale.** arXiv:2511.17190, 2025.
- Jian et al. **TRUST-SQL: Tool-Integrated Multi-Turn Reinforcement Learning for Text-to-SQL over Unknown Schemas.** arXiv:2603.16448, 2026.
- Biswal et al. **AgentSM: Semantic Memory for Agentic Text-to-SQL.** arXiv:2601.15709, 2026.
- Kim et al. **A Semantic-Layer-Mediated Agent for Natural Language to SQL over Heterogeneous Enterprise Databases.** arXiv:2606.31041, 2026.
- Gao et al. **SemanticAgent: A Semantics-Aware Framework for Text-to-SQL Data Synthesis.** arXiv:2604.21414, 2026.
- Zhu et al. **ReViSQL: Achieving Human-Level Text-to-SQL.** arXiv:2603.20004, 2026.
- Wang et al. **Agentar-Scale-SQL: Advancing Text-to-SQL through Orchestrated Test-Time Scaling.** arXiv:2509.24403, 2025.
- Guo et al. **Rethinking Agentic Workflows: Evaluating Inference-Based Test-Time Scaling Strategies in Text2SQL Tasks.** arXiv:2510.10885, 2025.
- Li et al. **SWE-SQL: Illuminating LLM Pathways to Solve User SQL Issues in Real-World Applications.** arXiv:2506.18951, 2025.
- Dong et al. **PRACTIQ: A Practical Conversational Text-to-SQL Dataset with Ambiguous and Unanswerable Queries.** NAACL 2025 / arXiv:2410.11076.
- Ding et al. **AmbiSQL: Interactive Ambiguity Detection and Resolution for Text-to-SQL.** arXiv:2508.15276, 2025.
- Chen et al. **Reliable Text-to-SQL with Adaptive Abstention.** arXiv:2501.10858, 2025.
- Pourreza et al. **CHESS: Contextual Harnessing for Efficient SQL Synthesis.** arXiv:2405.16755, 2024.

## Imported foundations

- Ellis et al. **DreamCoder: Bootstrapping Inductive Program Synthesis with Wake-Sleep Library Learning.** PLDI 2021, DOI:10.1145/3453483.3454080.
- Buneman, Khanna, Tan. **Why and Where: A Characterization of Data Provenance.** ICDT 2001, DOI:10.1007/3-540-44503-X_20.
- Green, Karvounarakis, Tannen. **Provenance Semirings.** PODS 2007.
- Meliou et al. **The Complexity of Causality and Responsibility for Query Answers and Non-Answers.** PVLDB 2010; arXiv:1009.2021.
- Horn, Perera, Cheney. **Incremental Relational Lenses.** arXiv:1807.01948, 2018.
- Doyle. **A Truth Maintenance System.** Artificial Intelligence 12(3), 1979.
- de Kleer. **An Assumption-Based TMS.** Artificial Intelligence 28(2), 1986.
- Ratner et al. **Snorkel: Rapid Training Data Creation with Weak Supervision.** arXiv:1711.10160, 2017.
- Adams & MacKay. **Bayesian Online Changepoint Detection.** arXiv:0710.3742, 2007.
- Snodgrass & Ahn et al. temporal database literature: valid time, transaction time, and bitemporal models.
- Gupta, Mumick, Subrahmanian. **Maintaining Views Incrementally.** SIGMOD 1993, DOI:10.1145/170036.170066.
- Yadav et al. **Enzyme: Incremental View Maintenance for Data Engineering.** arXiv:2603.27775, 2026.
- Willsey et al. **egg: Fast and Extensible Equality Saturation.** POPL 2021 / arXiv:2004.03082.
- Tishby, Pereira, Bialek. **The Information Bottleneck Method.** arXiv:physics/0004057.
- Dual-control literature: Feldbaum; modern MPC surveys on simultaneous control and active uncertainty learning.
