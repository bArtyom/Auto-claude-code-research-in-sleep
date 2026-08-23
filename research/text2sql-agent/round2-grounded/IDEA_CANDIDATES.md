# Round 2 — Literature-Grounded Idea Candidates

> **Role in ARIS-style workflow:** `idea-creator`  
> **Input:** `LITERATURE_LANDSCAPE.md`  
> **Rule:** Every idea below must identify its foreign mechanism, nearest Text2SQL prior art, the actual novelty wedge, and a cheap falsification experiment.

## Scoring

Each candidate receives a qualitative score on five axes:

- **N** — novelty after nearby-literature check
- **F** — falsifiability / experiment cleanliness
- **I** — likely scientific impact
- **E** — engineering burden for a first paper
- **R** — risk of collapsing into already-known Text2SQL work

`S > A > B > C`; for E and R, lower is better.

---

# Idea 1 — ProvSQL-Repair: Provenance-Guided Result-to-SQL Repair

## Foreign mechanism

Database provenance, query causality/responsibility, reverse data management, relational lenses.

## Problem

Most interactive Text2SQL asks the user to clarify *before* or *during* SQL generation. But real users often discover the problem only after seeing the result:

- “this customer should not be in the table”;
- “this row is missing”;
- “this total is too high because refunds should be excluded”;
- “this cell should be 14, not 17.”

Today the user typically has to translate that observation back into a natural-language bug report. That throws away extremely structured supervision.

## Core method

Treat result corrections as **denotational constraints**.

```text
Question + candidate SQL
            ↓
         Result R
            ↓
User marks:
  unwanted tuple / missing tuple / wrong cell
            ↓
Why / why-not provenance
+ query causality / responsibility
            ↓
localize responsible base tuples + operators
            ↓
minimal semantic repair search
            ↓
re-run and check side effects
```

For an unwanted row, identify which join/filter/aggregation decisions and source tuples are responsible. For a missing row, derive why-not conditions. The LLM proposes a small semantic edit constrained by the provenance slice instead of rewriting the whole query.

## Nearest prior art

- BIRD-CRITIC / SWE-SQL: SQL issue debugging, but the issue is represented as a user problem description.
- PRACTIQ / AmbiSQL: clarification over question ambiguity, not output-level correction.
- Database provenance/causality: explains output tuples, but not agentic natural-language-to-query repair.
- Relational lenses: backward propagation of view edits, traditionally to source data rather than to an inferred query program.

## Novelty wedge

**New feedback modality:** `result correction → query repair`, rather than `clarification text → new SQL`.

The method is not “use provenance to explain SQL.” The claim is that provenance can turn one or a few output corrections into a high-bandwidth supervision signal for query synthesis.

## Minimal falsification

1. Start with 100 correct BIRD/Spider SQLs.
2. Inject one semantic mutation: join type, DISTINCT, predicate, temporal column, aggregation grain.
3. Execute mutant and gold query.
4. Automatically derive one discriminative feedback item: one unwanted row, missing row, or wrong aggregate cell.
5. Give baselines the same feedback:
   - LLM repair from natural-language description;
   - LLM repair from raw result diff;
   - provenance-localized repair.
6. Measure exact/execution repair success, edit distance, and collateral result damage.

### Kill condition

If provenance localization does not increase repair success at matched token budget or if most semantic mutations produce provenance too large to be useful, drop the idea.

**Score:** N=S, F=S, I=S, E=A, R=low.

---

# Idea 2 — DreamSQL: Wake-Sleep Semantic Library Induction

## Foreign mechanism

DreamCoder-style library learning, program refactoring, minimum-description-length intuition, e-graphs.

## Problem

A long-lived SQL agent sees recurring business semantics. Existing memory systems can retrieve previous traces, while semantic-layer systems rely on curated definitions. Neither makes **abstraction invention** the central learning problem.

## Core method

Alternate:

### Wake
Solve tasks using current primitive library `L_t`.

### Sleep
Compress successful semantic programs / SQL traces into recurring sub-programs and promote useful sub-programs into named operators.

Example discovered operators:

```text
active_customer(as_of)
recognized_revenue(period, currency)
latest_scd_snapshot(entity, date)
dedup_orders(key, event_time)
fiscal_quarter(date)
```

Then use `L_{t+1}` as the program vocabulary for future query reasoning.

The important artifact is not an embedding memory; it is a **small evolving DSL for one warehouse**.

## Nearest prior art

- AgentSM: semantic memory / structured trace reuse.
- semantic-layer-mediated agent: curated semantic layer + SMQ compiler.
- DreamCoder: automatic library induction in program synthesis, but not enterprise Text2SQL.

## Novelty wedge

Bridge the missing middle:

`raw repeated traces → automatically invented symbolic semantic layer`.

A useful paper should explicitly compare:

1. example retrieval;
2. trace memory;
3. manually curated semantic macros;
4. automatically induced macros.

## Minimal falsification

Construct a stream of tasks on one database with repeated latent motifs. Reveal tasks in chronological batches. After every 50 tasks, run a sleep phase and induce candidate abstractions.

Measure:

- future task accuracy;
- tokens/tool calls;
- abstraction reuse frequency;
- compression ratio of the solved-program corpus;
- negative transfer when a learned abstraction is too broad;
- human semantic coherence of induced operators.

### Kill condition

If induced abstractions do not outperform simple retrieval or if the useful library merely memorizes table/column names without compositional reuse, the thesis fails.

**Score:** N=S, F=A, I=S, E=B/A, R=medium-low.

---

# Idea 3 — TimeSQL-Memory: Bitemporal Semantic Memory with Drift Detection

## Foreign mechanism

Bitemporal databases + Bayesian online change-point detection + truth maintenance.

## Problem

Semantic memory usually stores a fact as if it were timeless:

`revenue := net_amount`

Enterprise semantics are not timeless. A company can redefine a KPI, migrate a fact table, change fiscal calendars, or alter its canonical source.

Worse, there are two different times:

- **valid time:** when the definition is true for the business;
- **knowledge time:** when the agent learned or corrected it.

## Core method

Represent each semantic memory fact as:

```yaml
claim: recognized_revenue uses net_recognized_amount
valid_from: 2025-01-01
valid_to: null
known_from: 2025-02-12
known_to: null
justifications:
  - dbt model commit ...
  - finance dashboard ...
dependents:
  - macro/revenue_by_customer
  - memory/query-pattern-31
```

Monitor evidence streams with an online change detector. When a possible semantic regime shift occurs, quarantine affected claims and use dependency-directed invalidation rather than clearing the entire memory.

## Nearest prior art

- AgentSM: semantic memory, but not bitemporal semantics as its main object.
- generic continual-memory research: memory evolution, but not tied to database valid/transaction-time semantics.
- temporal databases / TMS: mature foundations but not agentic Text2SQL.

## Novelty wedge

**Historical semantic correctness** and **drift-safe reuse**.

The target question is not just “does memory help?” but:

> Can an agent answer both current and historical business questions correctly when definitions change over time, while avoiding stale-memory propagation?

## Minimal falsification

Build a synthetic query stream with hidden semantic regime changes at known points:

- metric definition changes;
- canonical date field changes;
- join bridge changes;
- business alias changes.

Compare:

- no memory;
- static memory;
- recency-weighted memory;
- bitemporal memory;
- bitemporal + change-point detector + TMS invalidation.

Metrics:

- stale-belief error rate;
- drift detection delay;
- false invalidation;
- recovery cost;
- accuracy on “current” vs “as-of historical” queries.

### Kill condition

If simple recency weighting performs equivalently or if realistic semantic changes cannot be detected from available evidence, the additional machinery is unjustified.

**Score:** N=S, F=S, I=S, E=A, R=low.

---

# Idea 4 — AutoSemanticView: A Self-Materializing Database Agent

## Foreign mechanism

Materialized view selection, incremental view maintenance, online workload optimization.

## Problem

An agent repeatedly solves the same semantic subproblem:

`dedupe events → latest snapshot → active customer → recognized revenue`

A memory system makes reasoning cheaper, but still asks the agent to reason. A database system would often **materialize** a repeated useful computation.

## Core method

The agent maintains candidate semantic assets:

- verified views;
- dbt models;
- metrics;
- macros;
- reusable join contracts.

For candidate asset `v`, estimate:

```text
Utility(v)
= expected future reasoning saved
+ expected DB execution saved
+ expected semantic-error reduction
- maintenance cost
- storage cost
- drift risk
```

When utility is positive, create/materialize the asset. Maintain it incrementally and retire it when workload or semantics drift.

## Novelty wedge

The agent does not merely **adapt to** the data environment; it **rewrites the interface** so future Text2SQL becomes easier.

This creates a long-horizon optimization problem connecting agent learning to classical database physical/logical design.

## Minimal falsification

Replay a repeated analytics workload for 500–2000 tasks over evolving data.

Compare:

- stateless Text2SQL;
- semantic memory;
- fixed hand-curated semantic layer;
- adaptive semantic materialization.

Measure cumulative:

- task success;
- LLM tokens;
- DB execution cost;
- asset maintenance cost;
- number of reusable semantic assets;
- semantic regressions after schema changes.

### Kill condition

If memory retrieval captures nearly all gains without changing the environment, or asset maintenance dominates saved reasoning, the idea is not worthwhile.

**Score:** N=S/A, F=A, I=S for DB/systems venues, E=B, R=low.

---

# Idea 5 — SnorkelSQL: Weakly Supervised Semantic Verification

## Foreign mechanism

Data programming / Snorkel weak supervision.

## Problem

Semantic verification in enterprise SQL often lacks gold SQL. But there are many weak signals:

- SQL executes;
- dbt tests pass;
- PK/FK constraints are respected;
- output grain matches expected entity;
- result is stable under safe perturbations;
- lineage reaches canonical source;
- prior analyst query agrees;
- independent LLM judge accepts;
- row count / duplication alarms are quiet.

Each signal is noisy and correlated.

## Core method

Implement each signal as a labeling function `λ_i(SQL, task) ∈ {correct, wrong, abstain}`. Learn source accuracies and dependencies from unlabeled candidates plus a very small expert calibration set. Produce a probabilistic semantic correctness score.

## Nearest prior art

- SemanticAgent: explicit semantic diagnosis during synthesis.
- ReViSQL: high-quality expert-verified data.
- execution-guided selection / majority voting.
- Snorkel: general weak supervision, not SQL-specific verification.

## Novelty wedge

The claim is **enterprise-local verifier adaptation from heterogeneous weak tests**, not another LLM critic.

## Minimal falsification

Generate correct/mutated SQL pairs across 3–5 databases. Hide most gold labels. Compare:

- simple majority of signals;
- learned weak-supervision model;
- LLM judge;
- small supervised classifier with the same expert-label budget.

Evaluate silent-error AUROC, expected calibration error, and expert-label efficiency.

### Kill condition

If signal correlations make the label model unstable or a small LLM judge dominates at equal cost, stop.

**Score:** N=A, F=S, I=A, E=A, R=medium.

---

# Idea 6 — CauseAlign-SQL: Causal Responsibility as a Semantic Alignment Signal

## Foreign mechanism

Query causality and responsibility.

## Core intuition

A correct answer to “How many unique customers bought X?” should be causally sensitive to customer-level evidence in predictable ways. A wrong query that counts orders may be overly sensitive to duplicated order rows.

For each candidate query, compute approximate causal responsibility of input tuple classes / schema entities for selected outputs. Compare this **causal signature** with the intended semantic entities extracted from the question.

## Example

Question intent:

```text
entity = customer
measure = unique count
```

Candidate A's output is highly affected by duplicating an order tuple. Candidate B's `COUNT(DISTINCT customer_id)` is invariant. This causal mismatch is evidence against A.

## Novelty wedge

Move beyond syntactic question-SQL alignment to **question ↔ result-causality alignment**.

## Minimal falsification

Build controlled transformations that duplicate/delete input tuple classes. Test whether causal-signature mismatch predicts semantic mutations better than execution success, LLM confidence, or static AST features.

### Kill condition

If causality computation is too expensive or signatures fail to distinguish common semantic error families, use causality only inside Idea 1 rather than as a standalone paper.

**Score:** N=A/S, F=A, I=A, E=B, R=low.

---

# Idea 7 — DualSQL: Cross-Task Dual Control for Database Agents

## Foreign mechanism

Dual adaptive control / active system identification.

## Problem

TRUST-SQL and AutoLink actively explore metadata for the current query. A long-lived agent should also value information for *future* queries.

## Core method

When choosing a probe `a_t`, optimize:

```text
current_task_value(a_t)
+ λ * expected_future_warehouse_information(a_t)
- cost(a_t)
```

A slightly more expensive probe may be optimal today if it resolves a reusable unknown join relationship that will benefit many later tasks.

## Novelty wedge

**Current-query POMDP → workload-level dual control / cumulative regret.**

## Minimal falsification

Create query streams with recurring latent schema concepts. Compare myopic active exploration with dual-control exploration under the same total probe budget.

Metric: cumulative error/probe cost over stream, not single-query EX.

### Kill condition

If future-query benefits disappear under modest task diversity, or if a simple cache gives the same gain, downgrade.

**Score:** N=A, F=A, I=A/S, E=B, R=medium-low.

---

# Idea 8 — EGraph-SQL: Equality-Class Reconciliation before Ensemble Selection

## Foreign mechanism

Equality saturation / e-graphs.

## Problem

Parallel Text2SQL scaling creates many candidate strings. But many differ only through safe equivalences:

- join association under valid conditions;
- CTE vs subquery;
- predicate reordering;
- equivalent aliases;
- some aggregate rewrites.

Voting counts surface multiplicity as independent evidence.

## Core method

Parse candidates into relational/SQL IR, apply a deliberately conservative rewrite set, and form e-classes. Spend verifier budget per e-class, not per SQL string.

Define:

```text
surface diversity
vs
semantic e-class diversity
```

Then test whether e-class diversity is a better predictor of Pass@K and uncertainty.

## Nearest prior art

Multi-generator ensembles (XiYan-style), Agentar-Scale-SQL, DivSkill-SQL; equality saturation outside Text2SQL.

## Novelty wedge

**De-duplicate reasoning hypotheses semantically before selection.**

## Minimal falsification

Take existing multi-candidate outputs. Build 20–40 sound SQL rewrite rules. Measure:

- candidate compression ratio;
- verifier calls saved;
- selector accuracy at fixed budget;
- false merges (must be near zero).

### Kill condition

If SQL bag/null semantics make safe equivalence too narrow to reduce candidate redundancy meaningfully, stop.

**Score:** N=A, F=S, I=A, E=A/B, R=low.

---

# Idea 9 — IB-Warehouse: Minimal Sufficient Persistent Warehouse State

## Foreign mechanism

Information Bottleneck / sufficient statistics.

## Problem

Per-query retrieval repeatedly reconstructs relevant context. Full warehouse memory accumulates noise. The middle object should be a compressed state that preserves what matters for a workload distribution.

## Core method

Learn `Z = compress(warehouse evidence, history)` while optimizing:

```text
min I(Z; raw_warehouse_history)
max I(Z; future_query_success)
```

Practical `Z` could be a sparse symbolic graph plus learned embeddings:

- entities;
- canonical joins;
- metrics;
- dangerous exceptions;
- temporal validity.

## Novelty wedge

Persistent **workload-level context compression**, not top-k retrieval for each query.

## Minimal falsification

Long query stream; constrain persistent state to fixed byte/token budgets. Compare full history, vector memory, top-k retrieval, hand semantic layer, and bottleneck-trained state.

### Kill condition

If gains disappear once strong retrieval is included, this is merely another retriever.

**Score:** N=A, F=A, I=A, E=B, R=medium.

---

# Idea 10 — GroupTest-Schema: Sparse Relevant-Schema Discovery by Pooled Tests

## Foreign mechanism

Adaptive group testing / sparse recovery.

## Hypothesis

A query typically depends on a sparse set of schema objects among thousands. Instead of testing columns one by one, issue **pooled semantic tests** over groups of schema objects and recursively split only promising groups.

Possible pooled tests:

- batch metadata search over a semantic community;
- ask whether any member of a table cluster contains values matching an entity;
- aggregate lineage reachability tests;
- combined catalog summaries.

## Research contribution

Study whether relevant-schema discovery can approach sublinear probe complexity under realistic sparse-support assumptions.

## Minimal falsification

Large schema (>1000 columns) with gold relevant support. Compare greedy top-k, AutoLink-like expansion, and hierarchical pooled search by recall vs number of metadata tokens/probes.

### Kill condition

If the database/tool interface cannot implement pooled tests without simply reading the entire group, the analogy is artificial.

**Score:** N=A, F=A, I=B/A, E=A, R=medium-high.

---

# Idea 11 — JustificationGraph-SQL: Truth-Maintenance Memory

## Foreign mechanism

Doyle TMS / de Kleer ATMS.

## Core method

Every semantic memory claim stores:

- its supporting evidence;
- assumptions;
- dependent memories/macros;
- alternative contexts.

When evidence changes, the agent performs dependency-directed invalidation and can keep multiple incompatible assumption sets alive until a query forces resolution.

## Novelty assessment

As a **standalone** idea this is weaker because our previous round already proposed belief revision, and long-lived memory is a growing agent research theme. It becomes much stronger as the repair substrate for TimeSQL-Memory.

## Minimal falsification

Plant one wrong high-centrality semantic claim and compare contamination/recovery under append-only memory vs TMS invalidation.

**Score:** N=B/A standalone, F=S, I=A component, E=A, R=medium.

---

# Idea 12 — LensSQL: Bidirectional Text2SQL through Result Editing

## Foreign mechanism

Bidirectional transformations / relational lenses.

## Core method

Expose the output as an editable semantic surface. User operations:

- delete an incorrect row;
- pin a missing row that must appear;
- change expected grouping;
- mark two rows as “should be one entity”;
- set a target aggregate value/range.

The system computes a minimal change to the **query semantics**, not to source data, satisfying the new view constraint.

## Relationship to Idea 1

LensSQL is the interaction abstraction; ProvSQL-Repair is the provenance/causality implementation. They can be one paper or separated:

- systems/HCI paper: bidirectional result-editing interface;
- DB/ML paper: provenance-constrained synthesis algorithm.

## Kill condition

If realistic users cannot express useful corrections via result edits, focus on row/cell annotations only.

**Score:** N=S/A, F=A, I=S for interactive systems, E=B, R=low.

---

# 13. Ranked shortlist before novelty audit

| Rank | Idea | N | F | Impact | First-pilot burden | Main risk |
|---:|---|:---:|:---:|:---:|:---:|---|
| 1 | ProvSQL-Repair / Result Lens | S | S | S | A | provenance explosion / aggregate feedback |
| 2 | DreamSQL library induction | S | A | S | A/B | learned macros may overfit one warehouse |
| 3 | TimeSQL-Memory | S | S | S | A | simple recency baseline may be enough |
| 4 | AutoSemanticView | S/A | A | S | B | systems engineering + maintenance cost |
| 5 | SnorkelSQL verifier | A | S | A | A | weak signals may be too correlated |
| 6 | CauseAlign-SQL | A/S | A | A | B | causality cost / weak semantic signature |
| 7 | EGraph-SQL | A | S | A | A/B | conservative rewrite coverage |
| 8 | DualSQL | A | A | A/S | B | future-task recurrence assumptions |
| 9 | IB-Warehouse | A | A | A | B | collapses into better retrieval |
| 10 | GroupTest-Schema | A | A | B/A | A | pooled-test primitive may be artificial |
| 11 | JustificationGraph-SQL | B/A | S | A component | A | overlaps belief-revision memory |
| 12 | LensSQL interface | S/A | A | S | B | user interaction realism |

The next stage should aggressively try to **kill** these candidates rather than promote them automatically.
