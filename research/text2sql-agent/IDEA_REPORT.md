# Text2SQL × Agent Research Idea Landscape (2026-08)

> Status: exploratory research memo. This document deliberately optimizes for idea breadth first, then identifies a small number of falsifiable research bets.

## 0. Executive thesis

Text2SQL is moving from **single-shot semantic parsing** toward **interactive data-work agents**. The strongest opportunity is no longer “generate one better SQL string”; it is to build an agent that can **acquire missing evidence, manipulate a database/workspace, test hypotheses, ask targeted questions, recover from failures, and know when it is uncertain**.

The clearest research wedge is:

**Turn Text2SQL into an evidence-gated agentic program synthesis problem.**

Instead of optimizing only `P(SQL | question, schema)`, optimize a trajectory:

`question → information acquisition → schema/value hypotheses → candidate programs → execution probes → semantic tests → repair / clarification → final answer + calibrated evidence`.

This aligns with the direction exposed by current benchmarks. Spider 2.0 contains enterprise-scale schemas, multiple SQL dialects and workflow tasks; BIRD-Interact explicitly evaluates conversational and agentic interaction; CHESS showed that specialized agents for retrieval/schema selection/generation/testing already produce large gains and efficiency improvements. The remaining gap is that these systems usually have relatively fixed pipelines rather than a learned or principled **policy over tool use, evidence, uncertainty, and interaction**.

## 1. Why this is timely

### 1.1 Benchmark shift: SQL string → workflow

Spider 2.0 describes 632 enterprise workflow problems, with databases often exceeding 1,000 columns, multiple engines (BigQuery, Snowflake, SQLite/DuckDB variants), long contexts and multi-query operations. Its DBT setting is explicitly a repository-level code-agent task rather than classic text-to-SQL.

BIRD has simultaneously expanded into BIRD-Interact, LiveSQLBench and BIRD-CRITIC. BIRD-Interact reports very low success rates even for frontier models and highlights **Interaction-Time Scaling**: performance can improve when models are allowed longer interaction trajectories. This is strong evidence that the key research variable is becoming *how the model acts over time*, not only raw generation quality.

### 1.2 Architectural shift: monolithic prompting → specialist systems

CHESS decomposes the problem into Information Retriever, Schema Selector, Candidate Generator and Unit Tester agents. It reports that schema selection can reduce tokens by roughly 5× while improving accuracy, and that the full system reaches 71.10% on BIRD while using substantially fewer LLM calls than a leading proprietary method at the time.

XiYan-SQL pushes another axis: multiple schema filters/generators plus a learned selector. Its central lesson is that **candidate diversity + selection** can outperform trusting one decoding path.

OmniSQL attacks the data axis: synthesizing 2.5M examples across more than 16k synthetic databases and training compact open models. This suggests a natural hybrid research direction: use specialized Text2SQL models as cheap proposal engines, while an agentic controller spends frontier-model budget only when evidence is insufficient.

## 2. Failure taxonomy worth designing around

A useful agent should explicitly classify failures before trying to repair them.

| Failure class | Example | Agent action |
|---|---|---|
| Intent ambiguity | “top customers” = revenue or margin? | ask user or infer from organization glossary with confidence threshold |
| Schema ambiguity | multiple `customer_id` variants | inspect lineage, PK/FK, comments, sample values |
| Entity/value mismatch | “NYC” stored as `New York` | value search / fuzzy lookup |
| Hidden business semantics | “active” encoded by several flags | retrieve docs, dbt models, metrics definitions |
| SQL syntax/dialect | QUALIFY / DATE_DIFF differences | dialect-aware compile / repair |
| Semantic error with valid execution | query runs but computes wrong cohort | metamorphic tests / counterexample probes |
| Data-quality trap | duplicates, nulls, late-arriving facts | profiling probes / invariants |
| Multi-step workflow | intermediate temp tables needed | plan and execute program, not one SQL |
| Cost / latency blow-up | full scan in Snowflake | EXPLAIN / dry run / budget-aware rewrite |
| Uncertainty | several candidates produce plausible results | selective prediction, ask or abstain |

The important distinction is **execution-valid ≠ semantically correct**. A research system should therefore treat execution as one signal among several, not the terminal oracle.

## 3. Idea space

## Idea A — Evidence-Gated SQL Agent (EGSA)

### Core idea

Every irreversible decision (schema commitment, join path, aggregation semantics, final SQL) must be backed by an explicit evidence object. The controller can only finalize when all required evidence gates pass.

Possible evidence objects:

- schema evidence: relevant table/column + reason
- value evidence: observed samples supporting entity mapping
- join evidence: FK/lineage/cardinality probe
- semantic evidence: natural-language unit tests and metamorphic relations
- execution evidence: syntax/engine success
- cost evidence: EXPLAIN / bytes scanned / timeout
- uncertainty evidence: candidate agreement or calibrated verifier score

The agent state becomes a small provenance graph rather than a chat transcript.

### Novelty hypothesis

Most Text2SQL agent systems use tool calls but do not make **evidence completeness** a first-class termination condition. A strict evidence ledger may improve reliability and auditability, especially on Spider 2.0 / BIRD-Interact, while reducing useless retries.

### Falsifiable prediction

At matched token/tool budget, evidence-gated termination yields higher semantic success and lower silent-error rate than ReAct-style “execute until it looks right”.

### Why it fits ARIS

ARIS already encodes evidence gates, reviewer independence, run states and multi-stage artifacts. Text2SQL can become a domain-specific testbed for the same philosophy.

---

## Idea B — Active Schema Exploration as a POMDP

### Core idea

Treat the database as partially observed. At each step the agent chooses an information-gathering action:

- inspect schema neighborhood
- retrieve documentation
- sample column values
- estimate cardinality
- inspect PK/FK / lineage
- run tiny diagnostic SQL
- ask the user a clarification question

The objective is not merely final accuracy; it minimizes expected **information acquisition cost + generation cost + execution cost + error risk**.

### Research contribution

Define a learned or heuristic policy that decides *what to inspect next* based on posterior uncertainty. Compare against fixed retrieval pipelines and full-schema prompting.

### Strong variant

Train a small “exploration policy” model using trajectories generated by expensive frontier agents, then distill it. This could make industrial Text2SQL cheaper without losing reliability.

---

## Idea C — Counterexample-Guided SQL Synthesis (CEGIS-SQL)

### Core idea

Borrow from program synthesis. The agent proposes candidate SQL, then actively searches for **counterexamples** that distinguish candidates.

Examples:

- candidate A uses `COUNT(*)`, candidate B uses `COUNT(DISTINCT customer_id)` → probe duplicate distribution
- candidate A uses order date, B uses ship date → inspect mismatched rows
- candidate A inner-joins, B left-joins → count dropped entities

The verifier generates discriminative probes instead of generic “unit tests”. The next candidate is refined using the counterexample.

### Why promising

Text2SQL semantic errors often survive ordinary execution. Counterexamples target *behavioral disagreement*, which is closer to semantic verification than syntax repair.

### Evaluation metric

In addition to task success, measure:

- candidate disagreement resolved per probe
- semantic bugs found before finalization
- number of rows scanned / tool calls per resolved ambiguity

---

## Idea D — Clarification-First Text2SQL Agent

### Core idea

Learn when asking one concise question is more valuable than generating SQL.

The policy estimates expected value of clarification (EVoC):

`EVoC = expected error reduction − interaction cost`.

Only ask when ambiguity is decision-relevant. Questions should be contrastive, e.g. “By revenue, do you mean gross booked amount or recognized net revenue?” rather than “Can you clarify?”.

### Benchmark fit

BIRD-Interact provides direct motivation and an evaluation substrate. A good paper would focus on *selective interaction*: fewer but higher-value questions.

---

## Idea E — SQL Agent Memory That Stores Failures, Not Answers

### Core idea

Long-term memory should primarily store **repair patterns and environment facts** rather than past final SQL.

Memory item examples:

- `customer_status='A'` means active in this warehouse
- Snowflake timestamp column is UTC despite misleading name
- joining fact X to dimension Y requires bridge Z
- previous failure: direct join duplicates rows 4.3×

At inference, retrieve memories by structural similarity (schema motif, error class, dialect, business concept).

### Novelty hypothesis

Failure-centric memory is less leakage-prone and more transferable than nearest-neighbor SQL exemplars. It may especially help repeated use on one enterprise warehouse.

---

## Idea F — Budgeted Mixture-of-Agents for SQL

### Core idea

Use a cheap specialist Text2SQL model for 80–90% of proposal generation and route only ambiguous/hard cases to expensive frontier agents.

Possible stack:

1. compact model generates N diverse candidates
2. deterministic parser checks syntax / tables / columns
3. execution sandbox prunes failures
4. learned selector estimates semantic confidence
5. frontier reasoning agent is invoked only when candidates disagree or evidence gates fail

OmniSQL/XiYan-style models are natural proposal engines; an agent controller handles hard tail cases.

### Main metric

Accuracy–cost Pareto frontier, not accuracy alone.

---

## Idea G — Repository-Aware Text2SQL for dbt / analytics engineering

### Core idea

The “schema” is not only the live database. It includes:

- dbt models
- YAML descriptions/tests
- metric definitions
- SQL macros
- dashboards
- lineage graph
- issue history
- naming conventions

An agent retrieves and reasons across this repository, edits/creates SQL/dbt artifacts, runs tests, and repairs failures.

### Why high upside

Spider2-DBT makes this directly benchmarkable. It also moves the contribution from a narrow parser toward a code/data engineering agent, where repository context and iterative execution matter.

---

## Idea H — Adversarial SQL Reviewer / Debate

### Core idea

Separate generator from reviewer. The reviewer never sees the generator’s natural-language rationale, only question + environment + candidate SQL + allowed evidence. It must construct the strongest semantic rejection case.

A final adjudicator chooses between candidates using fresh evidence.

### ARIS connection

This maps almost exactly onto ARIS cross-model adversarial review. The interesting research question is whether reviewer independence reduces correlated semantic mistakes in SQL generation.

---

## Idea I — Metamorphic Verification for Text2SQL

### Core idea

Generate semantic relations that should hold even without ground-truth SQL.

Examples:

- adding a stricter filter cannot increase count
- partitioned counts should sum to total when categories are exhaustive and disjoint
- equivalent join reorderings should match
- date-window subset results should be bounded by superset window
- replacing an ID projection with count-distinct should match row cardinality under uniqueness assumptions

The agent automatically derives and executes such checks.

### Contribution

A verifier that catches semantic errors on *unlabeled* enterprise queries could be more impactful than another generation prompt.

---

## Idea J — Self-Improving SQL Agent via Trajectory Mining

### Core idea

Mine successful and failed production/benchmark trajectories to learn:

- which tool call was actually useful
- where the first wrong commitment occurred
- what evidence would have prevented it
- which repair operations generalize

Train a process reward model / verifier on these labels, then improve the controller.

Avoid using only final execution reward because it cannot localize semantic mistakes.

## 4. Top three research bets

### Bet 1: Evidence-Gated + Counterexample-Guided SQL Agent

Combine Idea A + C + I.

**Claim target:** an agent that must produce a compact evidence ledger and survive targeted counterexample probes reduces silent semantic failures at matched inference budget.

This is the most scientifically crisp: it defines a mechanism, has strong ablations, can work without training a giant new model, and maps well to modern benchmarks.

### Bet 2: Selective Clarification / Active Information Acquisition

Combine Idea B + D.

**Claim target:** learned value-of-information routing improves interactive success while asking fewer questions and inspecting less schema.

This is especially aligned with BIRD-Interact and could expose a genuinely new scaling axis beyond token budget.

### Bet 3: Repository-Aware dbt Agent with Failure Memory

Combine Idea E + G.

**Claim target:** structured memory of repository/environment failures enables persistent improvement on enterprise analytics tasks, outperforming stateless agents and answer-retrieval memories.

This has strong product relevance and Spider2-DBT fit, though evaluation infrastructure is heavier.

## 5. Recommended first project: EviSQL-Agent

Working title: **EviSQL-Agent: Evidence-Gated Counterexample-Guided Agents for Reliable Text-to-SQL**.

### Architecture

```text
User question
   ↓
Intent & ambiguity analyzer
   ↓
Evidence planner ───────────────┐
   ↓                            │
Schema/value/doc explorer       │
   ↓                            │
Candidate generator (diverse)   │
   ↓                            │
Static verifier                 │
   ↓                            │
Execution sandbox               │
   ↓                            │
Counterexample generator        │
   ↓                            │
Metamorphic / discriminative tests
   ↓                            │
Independent semantic reviewer  │
   ↓                            │
Evidence gate ── fail ──────────┘
   ↓ pass
Final SQL + answer + evidence ledger + confidence
```

### Termination rule

Do not finalize merely because SQL executes. Finalization requires:

1. referenced schema objects exist;
2. critical entity/value mappings are evidenced;
3. candidate ambiguity is below threshold or resolved by probes;
4. no high-severity counterexample remains;
5. SQL executes in target dialect (when execution is allowed);
6. cost guard passes;
7. reviewer confidence / calibrated gate passes.

If the gate cannot pass within budget, **ask a targeted clarification or abstain**.

## 6. Evaluation design

### Benchmarks

- **BIRD / BIRD-Interact / Mini-Interact**: classical + interaction-heavy evaluation.
- **Spider 2.0 Lite/Snow**: enterprise schemas and multi-dialect SQL.
- **Spider2-DBT**: repository-level agent behavior.
- **LiveSQLBench**: contamination-resistant, broad SQL spectrum.
- Optional: legacy Spider for sanity and comparability, but it should not be the main claim.

### Baselines

1. single-shot frontier LLM
2. ReAct-style SQL agent
3. fixed retrieve → generate → execute → repair pipeline
4. CHESS-like modular pipeline
5. multi-candidate + selector baseline inspired by XiYan
6. EviSQL without evidence gate
7. EviSQL without counterexample probes
8. EviSQL without independent reviewer

### Metrics

Primary:

- benchmark Success Rate / execution-based task score
- silent semantic error rate
- interaction success rate

Efficiency:

- tokens
- LLM calls
- DB tool calls
- rows/bytes scanned when available
- wall-clock latency

Agent quality:

- clarification precision: fraction of asked questions that change the final decision
- evidence utilization: fraction of collected evidence referenced by a gate decision
- failure localization accuracy
- abstention calibration / risk-coverage curve

### Critical ablations

- evidence ledger vs free-form scratchpad
- discriminative counterexample probes vs generic execution repair
- independent reviewer vs same-agent self-critique
- N candidate diversity levels
- full schema vs active schema acquisition
- fixed budget vs adaptive budget
- with/without clarification action

## 7. Minimum viable pilot

A useful first pilot does **not** require model training.

### MVP components

- SQLite/DuckDB sandbox
- schema explorer
- value sampler
- 3-candidate SQL generator
- SQL parser/static checker
- execution harness
- counterexample probe generator
- evidence ledger JSON
- final gate

Run on a 50–100 example subset of BIRD/Spider-like tasks and manually label *why* each baseline failure happened.

### Fast falsification test

Take 30 examples where two independently generated SQL candidates both execute but disagree. Ask the verifier to generate one discriminative probe. Measure how often the probe identifies the semantically correct candidate.

If this rate is barely above random, CEGIS-SQL is weak. If it is high, the mechanism deserves a full paper.

## 8. Risk register

| Risk | Mitigation |
|---|---|
| Execution result leaks gold-like information | strictly define allowed observations; never compare to gold output during inference |
| LLM “unit tests” are circular | use behavioral/metamorphic probes and independent reviewer |
| More tools simply mean more compute | report matched-budget Pareto curves |
| Evidence ledger becomes verbose theater | gate must reference machine-checkable evidence IDs |
| Benchmark-specific hacks | test across BIRD + Spider2 + LiveSQLBench dialects |
| Agent asks too many questions | explicit interaction cost + EVoC threshold |
| Data access/privacy | support metadata-only/value-redacted modes |
| Same-model verifier correlated errors | cross-family reviewer where feasible; report same-family as separate ablation |

## 9. Relationship to ARIS

ARIS already has concepts that are unusually appropriate for this research:

- executor/reviewer separation
- evidence-based stage gates
- resumable run state
- multi-agent fan-out
- experiment integrity audits
- artifact contracts

A future ARIS domain skill could be `/text2sql-agent-research`, chaining literature search, benchmark setup, agent implementation, failure mining and adversarial evaluation. Importantly, benchmark execution should adopt ARIS’s rule that the executor must not certify its own correctness.

## 10. Source notes

Key public references used for this memo:

- Spider 2.0 official site: https://spider2-sql.github.io/
- BIRD benchmark / BIRD-Interact / LiveSQLBench news: https://bird-bench.github.io/
- CHESS: Contextual Harnessing for Efficient SQL Synthesis, arXiv:2405.16755 — https://arxiv.org/abs/2405.16755
- OmniSQL: Synthesizing High-quality Text-to-SQL Data at Scale, arXiv:2503.02240 — https://arxiv.org/abs/2503.02240
- XiYan-SQL: A Novel Multi-Generator Framework For Text-to-SQL, arXiv:2507.04701 — https://arxiv.org/abs/2507.04701

## 11. Next decision

If only one direction is pursued first, choose **EviSQL-Agent (Evidence-Gated + Counterexample-Guided verification)**. It has the best combination of novelty, falsifiability, compatibility with existing models, and fit with ARIS’s cross-model assurance philosophy.
