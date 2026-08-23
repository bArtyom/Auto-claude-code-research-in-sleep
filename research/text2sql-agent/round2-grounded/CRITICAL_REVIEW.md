# Round 2 — Critical Review of the Surviving Ideas

> **Role in ARIS-style workflow:** `research-review`  
> **Review status:** **PROVISIONAL / SAME-MODEL INTERNAL RED TEAM**. This document is deliberately adversarial, but it is **not** an independent cross-model review receipt and must not be treated as ARIS acceptance. A submission-grade run should send the final proposal/artifacts cold to an independent reviewer family and record the returned identity/trace.

## 0. Reviewer mandate

The job here is not to make the ideas sound exciting. The job is to identify the simplest explanation under which each idea is incremental, unnecessary, untestable, or likely to lose to a strong baseline.

The literature audit already killed one attractive idea (SnorkelSQL) and weakened several others. This review continues that pressure.

---

# 1. DreamSQL — Automatic Semantic Library Induction

## Strongest rejection case

This may be **program compression dressed up as database semantics**.

SQL workloads naturally contain repeated syntax: date filters, joins, deduplication idioms, CTEs. A library learner can achieve impressive compression by inventing macros that are syntactically common but semantically trivial. That does not imply better Text2SQL reasoning.

A second risk is that modern semantic-memory systems and execution-grounded mappings already capture most reusable information. If AgentSM/GATE-style memory plus retrieval reaches the same held-out performance as induced abstractions, the extra symbolic machinery has no scientific necessity.

A third risk is **warehouse-specific overfitting**. If the induced operators encode physical table names and fragile implementation details, they will not compose under schema evolution and will be little more than named snippets.

Finally, evaluation is nontrivial. Compression ratio can improve while semantic accuracy stays flat. Human-readable macro names can be generated post hoc and may exaggerate semantic coherence.

## What would convince a skeptical reviewer

DreamSQL needs evidence on **three distinct axes**:

1. **Compression:** the library actually shortens the representation of a workload.
2. **Compositional reuse:** held-out tasks are solved by recombining learned abstractions in new ways, not by retrieving near-duplicates.
3. **Reasoning benefit:** at matched memory/token/tool budget, the induced library improves success or reduces search relative to retrieval-based memory.

A strong test should include **anti-memorization splits**: hold out combinations of motifs while retaining the motifs individually during training. Example:

- training contains `active_customer`, `recognized_revenue`, `fiscal_period` in separate tasks;
- test requires composing all three in a novel query.

If the library merely memorizes whole query templates, it should fail this split.

## Recommended first implementation choice

Do **not** build the full DreamCoder machinery first. Use an existing fast library-learning idea such as Stitch-style top-down abstraction synthesis on a normalized semantic/relational IR. This isolates the scientific question—whether reusable abstractions exist—before investing in a wake-sleep neural architecture.

## Verdict

**KEEP — strongest high-upside idea, but only if evaluation is about compositional transfer, not macro compression.**

---

# 2. Result2SQL — Denotational Feedback for Interactive Text2SQL

## Strongest rejection case

The database community has long studied provenance, why-not provenance, bidirectional transformations, and query synthesis from examples. SPARTA already uses why-not provenance to refine LLM-generated queries. A reviewer could say:

> “You changed the source of the expected tuple from an automatically generated benchmark signal to a user click. That is an interface contribution, not a new algorithm.”

There is also a practical weakness: users often cannot identify a *correct missing row* if they do not already know the answer. Wrong aggregate cells are even harder; a user who knows the correct total may already have another trusted system.

For aggregation queries, provenance can explode. A wrong total may depend on millions of source tuples. Naive provenance does not automatically localize the faulty SQL operator.

Another concern is that an LLM receiving a simple natural-language correction (“exclude refunded orders”) may already repair the query perfectly. Provenance would add complexity without gain.

## What would convince a skeptical reviewer

The paper must show **feedback efficiency** and **locality**.

A useful formulation is:

> Given one sparse correction on the denotation, how much semantic error can be repaired while minimally changing outputs that the user did not mark as wrong?

Key metrics:

- repair success after 1, 2, 3 corrections;
- number of corrected tuples/cells needed;
- collateral damage: previously correct output tuples changed by the repair;
- semantic edit locality: how many SQL clauses/operators changed;
- latency/token cost.

The hardest and most valuable setting is not empty-result repair. It is **executable, plausible, partially correct SQL** where only a few outputs reveal the semantic bug.

## Essential baselines

1. LLM + text feedback only.
2. LLM + raw before/after result diff.
3. LLM + a random/exhaustive subset of source rows.
4. Provenance-localized repair.
5. If available, example-guided relational query synthesis adapted to the same feedback.

## User realism

Avoid assuming omniscient feedback. Generate several feedback regimes:

- **negative tuple:** “this row should not be present” — realistic.
- **positive tuple:** “this known customer should appear” — plausible for users with a concrete case.
- **pairwise equivalence:** “these two rows refer to the same entity” — realistic in dedup/entity tasks.
- **aggregate direction:** “this total is too high/low” — weaker supervision than exact target value.

If the method only works when users provide exact gold outputs, the interaction story is weak.

## Verdict

**KEEP — experimentally clean and publishable if framed as sparse denotational supervision + minimal collateral repair, not 'provenance for LLMs'.**

---

# 3. AutoSemanticView — Self-Materializing Semantic Infrastructure

## Strongest rejection case

This idea risks becoming an overengineered combination of:

- query/result caching;
- materialized-view selection;
- semantic layers;
- agent memory.

All four are established. A skeptical DB reviewer will ask which optimization problem is genuinely new and whether the LLM is necessary at all.

There is also an evaluation trap: if the workload contains heavy repetition, materialization obviously helps; if it does not, it will not. A hand-designed frequency threshold might outperform an elaborate agent policy.

Semantic maintenance is harder than physical view maintenance. If a business definition changes, a materialized semantic asset may silently become wrong. The cost model therefore needs semantic-risk terms that are measurable, not rhetorical.

## What would convince a skeptical reviewer

The paper needs to demonstrate **cross-layer value** that neither memory nor classic materialized-view selection alone captures.

Candidate decision:

```text
Should repeated semantic reasoning be promoted from
  transient trajectory
→ memory item
→ reusable symbolic abstraction
→ executable semantic asset?
```

The asset has two forms of amortization:

1. LLM reasoning amortization;
2. DB execution amortization.

The systems objective can explicitly combine both.

A strong baseline matrix:

- no memory/no materialization;
- vector or semantic memory only;
- classical materialized-view selection using workload frequency/cost;
- static human semantic layer;
- adaptive semantic asset policy.

## Minimal viable paper

Do not begin with automatic dbt writes in a production warehouse. First build an **offline workload replay simulator** in DuckDB/Postgres:

- repeated query stream;
- candidate abstractions/views;
- measured query runtime or optimizer cost;
- simulated LLM reasoning cost from recorded trajectories;
- controlled schema/semantic drift events.

Only if the offline utility curve is positive should the project move to live asset creation.

## Verdict

**KEEP — high systems upside, but probably a second paper after DreamSQL/Result2SQL.**

---

# 4. TimeSQL / Semantic-Drift Benchmark

## Strongest rejection case

Recent 2026 memory papers already use bitemporal memory and contradiction-resolution operators. A method paper centered on “valid time + transaction time for agent memory” will look late.

Synthetic drift can also be too easy. Renaming a column or switching one metric formula at a known time is not representative of messy enterprise evolution.

## What remains valuable

A **benchmark** can be compelling if it captures phenomena unique to analytics semantics:

- restated financial metric definitions;
- slowly changing dimensions and source migrations;
- retrospective corrections where the *historical* answer should change;
- definition changes where the historical answer should **not** change;
- late discovery of a definition that was valid earlier;
- conflicting authoritative sources with different validity intervals.

The benchmark should distinguish:

```text
What was true then?
What did the agent know then?
What does the agent know now about what was true then?
```

This is richer than recency weighting.

## Verdict

**KEEP as benchmark/infrastructure. Do not lead with bitemporal memory as algorithmic novelty.**

---

# 5. CauseAlign-SQL

## Strongest rejection case

“Duplicate a row and see whether the answer changes” is metamorphic testing. Calling it causal responsibility may add mathematical vocabulary without adding information.

Exact responsibility can also be computationally expensive, especially with aggregation and joins.

## Required proof-of-signal

Before writing an architecture, answer one empirical question:

> Across common executable semantic mutants, does a causal-responsibility signature distinguish wrong from correct SQL better than cheap metamorphic features and AST features?

If not, kill it.

## Verdict

**PROBE ONLY. No architecture work until the signal is demonstrated.**

---

# 6. DualSQL

## Strongest rejection case

A simple warehouse cache already gives future value to information collected today. Unless the system sometimes performs a probe that is **suboptimal for the current task but optimal for the future workload**, dual control is just active retrieval + caching.

## Required proof

Create environments where the distinction is observable. Example:

- current query can be solved cheaply without verifying a join relationship;
- verifying it now costs one extra probe;
- many future queries will need that relationship.

A dual-control policy should sometimes pay the extra cost now; a myopic policy should not.

Measure cumulative regret under varying recurrence rates. There should be a clear regime where dual control wins and a regime where it correctly becomes myopic.

## Verdict

**KEEP conditionally. The future-value term must change actual actions, not just ranking scores.**

---

# 7. EGraph-SQL

## Strongest rejection case

SQL equivalence is much harder than expression equivalence because of bag semantics, NULLs, nondeterminism, floating-point behavior, window functions, dialect quirks, and correlated subqueries.

A sound rewrite set may be too small to collapse enough candidates. An aggressive set risks catastrophic false equivalence.

## Cheap experiment

On a large archive of multi-candidate SQL outputs, implement only obviously safe canonicalizations and a small verified rewrite set. Measure candidate compression **before** building an e-graph infrastructure.

If conservative normalization already collapses most duplicates, e-graphs may be unnecessary. If it collapses almost none, the opportunity is small.

## Verdict

**KEEP as a cheap efficiency probe; likely workshop/systems component rather than flagship thesis.**

---

# 8. Cross-idea concern: benchmark contamination and false scientific progress

ReViSQL's result that a large fraction of inspected BIRD training annotations contain errors should change the experimental discipline for every proposal.

For pilots, use a small **expert-verified** subset or construct controlled semantic mutants from verified SQL. Otherwise a verifier/repair method may appear wrong because the label is wrong—or appear successful by reproducing annotation artifacts.

Every result table should separate:

- execution agreement with benchmark;
- expert semantic correctness;
- query/result stability under controlled tests.

---

# 9. Cross-idea concern: strong simple baselines

Rethinking Agentic Workflows and ReViSQL both caution against attributing gains to orchestration when model/data quality may explain them.

For every proposed method:

1. match total LLM tokens/calls;
2. include best-of-N/self-consistency where applicable;
3. include a stronger base model or specialist Text2SQL model if affordable;
4. report latency and DB tool calls;
5. pre-register a kill threshold before scaling the experiment.

An idea that only beats a weak one-shot prompt is not worth pursuing.

---

# 10. Final reviewer recommendation

## Primary paper bet A — DreamSQL

**Working title:** *Learning the Semantic Language of a Data Warehouse from Repeated Query Solving*

Core research question:

> Can a database agent automatically invent a compact library of reusable semantic programs whose composition improves future Text2SQL beyond example/trace retrieval?

Why this is attractive:

- strong conceptual difference from current memory;
- direct connection to program synthesis;
- long-horizon agent relevance;
- natural follow-on into self-materialized semantic infrastructure.

Main risk: abstraction learner discovers only syntax, not semantics.

---

## Primary paper bet B — Result2SQL

**Working title:** *Sparse Denotational Feedback for Interactive Text-to-SQL Repair*

Core research question:

> Can one or two user corrections on a query result localize and repair an executable semantic SQL error more efficiently than textual clarification or unconstrained regeneration?

Why this is attractive:

- very clean falsification;
- new interaction modality relative to most Text2SQL systems;
- strong database-theory foundations;
- easy controlled benchmark construction.

Main risk: nearby provenance/query-synthesis literature narrows novelty; aggregate feedback may be difficult.

---

## Systems paper bet C — AutoSemanticView

**Working title:** *When Should a Data Agent Turn Reasoning into Infrastructure?*

Core question:

> Under a repeated workload, when should a learned semantic abstraction be promoted into an executable, maintained semantic asset?

Main risk: classical database view selection plus cache may explain all gains.

---

# 11. Portfolio recommendation

Do **not** merge all three into one giant architecture initially.

Use a staged research program:

```text
Phase 1: Result2SQL
  learn how sparse feedback repairs individual programs

Phase 2: DreamSQL
  learn how many verified programs compress into reusable abstractions

Phase 3: AutoSemanticView
  learn when recurring abstractions should become maintained infrastructure
```

The papers can share a long-horizon thesis, but each retains one falsifiable mechanism.

## Acceptance status

`PROVISIONAL` — internal skeptical review only. Independent cross-model review still required for ARIS-style accepted status.
