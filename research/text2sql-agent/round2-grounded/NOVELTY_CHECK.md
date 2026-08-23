# Round 2 — Novelty Check and Idea Elimination

> **Role in ARIS-style workflow:** `novelty-check`  
> **Method:** targeted nearby-work searches after candidate generation.  
> **Policy:** finding close work is a success. We downgrade or kill ideas rather than moving the novelty goalposts.  
> **Status:** literature-based novelty audit, **not** patent-level or exhaustive novelty proof.

## 0. Executive result

The second-pass search changed the ranking materially.

Two candidates were hit by direct or near-direct prior art:

1. **SnorkelSQL / weakly supervised SQL verifier → KILL as standalone contribution.** SQLENS already aggregates noisy database/LLM error signals using weak supervision and learns labeling-function accuracies/correlations.
2. **Bitemporal memory as the core mechanism → DOWNGRADE.** Very recent 2026 work already applies bitemporal models to LLM-agent persistent memory. The surviving contribution must be specifically about *database semantic validity over time*, query-stream drift, and dependency-aware invalidation—not “we add bitemporal memory to an agent.”

A third candidate was partially anticipated:

3. **Provenance-guided repair → NARROW THE CLAIM.** SPARTA (ICLR 2026) already uses why-not provenance to localize predicates that cause empty results and feeds that information to an LLM for refinement. However, it uses provenance inside benchmark/query synthesis and derives expected tuples from intermediate results. Our surviving wedge is **human denotational correction of a generated Text2SQL result** (wrong/missing rows or cells) as a backward supervision channel, together with minimal query-semantic repair and collateral-damage constraints.

This is exactly why this stage matters: without the second literature pass, we would have overclaimed three ideas.

---

# 1. Candidate-by-candidate audit

## 1. ProvSQL-Repair / Result Lens

### Newly found close work

**SPARTA: Scalable and Principled Benchmark of Tree-Structured Multi-hop QA over Text and Tables** (Park, Kim, Han; ICLR 2026; arXiv:2602.23286) contains a provenance-based refinement loop. When a generated nested query returns no rows, it backs off predicates, obtains expected tuples from an intermediate non-empty subquery, runs why-not provenance, identifies a problematic predicate, and prompts the LLM to modify that clause.

Older foundations also reduce the novelty of “synthesize a relational query from examples”:

- **Example-Guided Synthesis of Relational Queries** (Thakkar et al., PLDI 2021) synthesizes relational queries from input-output examples.
- why-not provenance and reverse data management have long histories in database research.

### What claim does NOT survive

> “First system to use provenance for SQL refinement.”

False / indefensible.

### Surviving claim

> **Use end-user corrections on the *result of an NL-generated query* as a high-bandwidth interactive supervision channel, and combine provenance/causality with a minimal semantic edit objective to repair the Text2SQL program while constraining collateral output changes.**

Key distinctions to test explicitly:

1. feedback is provided by a user on output tuples/cells, not inferred automatically because the result is empty;
2. candidate SQL originated from an NL intent, so the repair must remain faithful to the question;
3. the system supports **unwanted**, **missing**, and ideally **wrong aggregate** feedback, not only empty-result predicates;
4. repair optimizes minimal semantic change and side-effect control;
5. evaluation measures **information efficiency of feedback**: how much query error can be repaired per corrected output item?

### Novelty confidence

**Medium-high**, after narrowing.

### Decision

**KEEP, but rename/focus:** `Result2SQL Repair` or `Denotational Feedback for Text2SQL`, with provenance as the enabling mechanism rather than the headline novelty.

---

## 2. DreamSQL: Wake-Sleep Semantic Library Induction

### Newly found close work

**GATE: Bootstrapping Semantic Layer from Execution for Text-to-SQL** (Lee, Kim, Hwang, 2026, arXiv:2606.05634) is important nearby work. GATE keeps grounding hypotheses open, uses execution observations to choose supported groundings, and stores the resulting groundings in reusable memory.

Also relevant:

- AgentSM stores reusable semantic programs from execution traces.
- curated semantic-layer systems already provide reusable high-level semantic objects.
- industrial/open-source projects are beginning to infer semantic layers automatically from warehouse metadata.

### What claim does NOT survive

> “First to automatically learn reusable semantics from execution.”

Too broad; GATE directly weakens it.

### Surviving claim

DreamSQL must make **abstraction invention and language growth** the irreducible mechanism:

```text
many solved programs
   ↓
refactor/compress repeated semantic subprograms
   ↓
invent a new symbolic operator
   ↓
rewrite old solutions in the expanded DSL
   ↓
use the new operator compositionally on future tasks
```

A memory entry such as “`active` maps to status='A' here” is not enough. The method must discover **higher-order reusable programs** that compress multiple solved tasks, e.g. `latest_valid_customer_snapshot(as_of)`.

### Necessary baseline set

- nearest-neighbor examples;
- AgentSM-like structured trace memory;
- GATE-style execution-grounded memory;
- fixed human semantic layer;
- DreamSQL induced library.

### Novelty confidence

**Medium-high**. Search found strong neighboring work on semantic memory and bootstrapping but not a clear Text2SQL analogue whose main objective is DreamCoder-style corpus compression / DSL invention.

### Decision

**KEEP as top candidate.** Do not sell “wake-sleep” by itself; sell **automatic semantic language induction from a query workload**.

---

## 3. TimeSQL-Memory: Bitemporal Semantic Memory + Drift

### Newly found direct adjacent work

Recent general-agent memory work already uses bitemporal semantics:

- **A Graph-Native Bitemporal Memory Store for Conversational AI Agents** (Niksarli & Baheti, arXiv:2607.26520) stores valid time and transaction time for agent memories.
- **TOKI: A Bitemporal Operator Algebra for Contradiction Resolution in LLM-Agent Persistent Memory** (Wang, arXiv:2606.06240) formalizes bitemporal contradiction-resolution operators.
- **Governed Persistent Memory** (Xu, arXiv:2608.12476) uses bitemporal state semantics plus provenance/lifecycle controls for long-horizon agent memory.

### What claim does NOT survive

> “Introduce bitemporal memory for LLM agents.”

Directly crowded by 2026 work.

### Surviving wedge

Database analytics introduces a domain-specific temporal problem that generic conversational memory does not solve:

1. business metric definitions themselves have valid intervals;
2. source-of-truth tables and join contracts evolve;
3. an “as-of 2024” query should use the 2024 semantic definition even if the agent learned the correction in 2026;
4. one changed semantic fact can invalidate many stored SQL abstractions and cached analyses.

Thus the contribution should be:

> **A benchmark and algorithm for semantic-regime drift in long-lived database agents, combining temporal validity, online change detection, and dependency-directed invalidation.**

Bitemporality becomes a representation choice, not the novelty claim.

### Novelty confidence

**Medium** for method; **medium-high** for a well-designed evolving-warehouse benchmark.

### Decision

**KEEP, DOWNGRADE from S-method to A/S-benchmark.** Strong as a long-horizon research program, less strong as a single “new memory architecture” paper.

---

## 4. AutoSemanticView: Self-Materializing Database Agent

### Nearby work found

- classical materialized-view selection and incremental view maintenance;
- 2026 systems such as **Enzyme** show continued industrial IVM relevance;
- semantic layers for AI agents are becoming mainstream;
- **GATE** bootstraps reusable execution-grounded semantic memory;
- projects such as “semlayer” advertise automatic semantic-layer inference from a warehouse.

### What remains distinct

The specific optimization problem appears different:

> At inference/runtime over a task stream, should the agent **create, maintain, merge, or retire an executable semantic asset** because doing so lowers expected future reasoning + query cost while preserving verified semantics?

This is closer to materialized view selection / database design than memory retrieval.

A rigorous contribution must include:

- workload-frequency model;
- semantic confidence requirement before materialization;
- maintenance cost and drift risk;
- retirement/invalidating policy;
- cumulative objective.

### Novelty confidence

**Medium-high**, especially for DB-systems framing. It becomes weak if the “materialized asset” is merely a YAML memory record.

### Decision

**KEEP.** Favor a DB/systems paper narrative over a generic LLM-agent paper.

---

## 5. SnorkelSQL: Weakly Supervised Semantic Verification

### Direct prior art found

**SQLENS: Fine-grained and Explainable Error Detection in Text-to-SQL** (OpenReview submission `CusEAujXDm`) already does the core proposed mechanism:

- collects heterogeneous database and LLM error signals;
- treats them as noisy labeling functions;
- uses weak supervision to learn signal accuracy/correlation;
- predicts SQL semantic correctness and produces error reports.

This is essentially the original SnorkelSQL proposal.

### Decision

**KILL as a standalone research idea.**

It remains useful as a **baseline or component** in another proposal, but we should not present weak-supervision aggregation itself as new.

### Lesson

A high-level cross-domain analogy (“Snorkel for SQL”) can sound novel while direct work already exists. Literature checking must happen before architectural investment.

---

## 6. CauseAlign-SQL: Causal Responsibility as Semantic Alignment

### Nearby work

Database causality/responsibility is mature; provenance is increasingly used with LLMs for refinement. However, targeted search did not reveal a clear Text2SQL method whose primary signal is:

`question-intended semantic entity/measure ↔ intervention/causal sensitivity signature of the query result`.

### Main novelty risk

Reviewers may say this is just metamorphic testing expressed in causal vocabulary.

To survive, the method must exploit an actual causal/responsibility computation that gives a signal not obtainable from generic perturbations—for example, ranking responsible tuple classes/operators and using those rankings for clause localization.

### Decision

**KEEP as exploratory A-tier, not a primary bet yet.** Run one cheap diagnostic experiment before investing.

---

## 7. DualSQL: Cross-Task Dual Control

### Nearby work

TRUST-SQL explicitly formulates unknown-schema Text2SQL as a POMDP, and AutoLink performs iterative autonomous schema exploration. Those are strong prior art for **within-task** information acquisition.

### Surviving wedge

The distinctive object is **cumulative regret across a query stream**:

- a probe can be locally unnecessary yet globally rational because it learns a reusable warehouse fact;
- action utility includes expected effect on future tasks;
- task distribution / recurrence becomes part of the environment.

### Novelty risk

If implementation reduces to “cache everything we discover,” dual-control terminology becomes decorative.

### Decision

**KEEP as A-tier only if an explicit future-value policy beats myopic exploration + cache.**

---

## 8. EGraph-SQL: Equality-Class Reconciliation

### Nearby work

Equality saturation is established in compilers/program synthesis. Text2SQL has extensive ensemble/test-time scaling work, including Agentar-Scale-SQL and complementary skill ensembles.

Targeted search did not surface a clear mainstream Text2SQL system that first builds conservative SQL equivalence classes with an e-graph and allocates verifier budget by semantic class.

### Risk

SQL equivalence is treacherous under bags, NULL, three-valued logic, nondeterminism, dialect differences, and floating point. A broad rewrite system will be unsound; a narrow one may produce too little compression.

### Decision

**KEEP as clean systems/efficiency experiment.** Novelty confidence medium-high; expected upside moderate.

---

## 9. IB-Warehouse: Minimal Sufficient Warehouse State

### Nearby work

Schema retrieval, semantic memory, and semantic layers are all crowded. Information Bottleneck itself is old and broad.

### Novelty risk

A reviewer may correctly summarize the method as “learn another compressed retriever.”

### Decision

**DOWNGRADE to B-tier** unless a precise theorem/objective connects persistent state size to future-query utility and the representation is demonstrably reusable across query classes.

---

## 10. GroupTest-Schema

### Nearby work

AutoLink / TRUST-SQL already attack large unknown schemas adaptively.

### Main concern

Classical group testing depends on a pooled-test primitive with a well-defined observation model. If our “pooled test” is simply “summarize 200 columns with an LLM,” the theoretical analogy is weak.

### Decision

**DOWNGRADE to B-tier moonshot.** Only revive if a realistic database operation can implement pooled evidence acquisition with sublinear information cost.

---

## 11. JustificationGraph-SQL / TMS memory

### Nearby work

General memory governance, bitemporal memory, contradiction resolution, plus our own previous belief-revision brainstorming substantially reduce standalone novelty.

### Decision

**MERGE into TimeSQL-Memory** as the dependency-invalidation mechanism. Do not pursue as a separate paper.

---

## 12. LensSQL

### Nearby work

Relational lenses and example-guided relational query synthesis are well established. This prevents a generic “edit output → synthesize query” novelty claim.

### Surviving wedge

The interesting setting is **interactive natural-language analytics**:

- initial SQL comes from NL intent;
- user performs a *small correction* on the observed denotation;
- system must preserve the original intent except where the feedback proves it wrong;
- repair should have minimal collateral effect;
- multi-turn corrections should converge efficiently.

### Decision

**MERGE with ProvSQL-Repair**. The combined paper should be framed as denotational feedback for Text2SQL, not as inventing bidirectional databases.

---

# 2. Revised ranking after novelty search

## Tier S — strongest surviving paper bets

### S1. DreamSQL — Automatic Semantic Library Induction

Why it survives:

- AgentSM = reuse traces;
- GATE = learn/store execution-grounded mappings;
- semantic-layer agent = consume curated semantics;
- DreamSQL = **invent reusable higher-level semantic programs that compress a workload and change the DSL available to future queries**.

Main experiment: does abstraction invention beat retrieval at matched memory/token budget on repeated compositional motifs?

---

### S2. Result2SQL — Denotational Feedback for Interactive Text2SQL

Why it survives:

- provenance-based query refinement exists;
- relational query synthesis from examples exists;
- but the proposed task interface is narrower and operationally different: **repair an NL-grounded SQL from sparse user corrections on its actual result**, with provenance-constrained localization and minimal side effects.

Main experiment: repair success per one corrected tuple/cell.

---

### S3. AutoSemanticView — Self-Materializing Semantic Infrastructure

Why it survives:

- memory/semantic-layer work mostly changes what the agent remembers/reads;
- materialization changes the **environment and future query interface**;
- classical DB work gives a principled cost/maintenance framework.

Main experiment: cumulative workload utility vs semantic memory and fixed semantic layer.

---

## Tier A — strong but either more crowded or more conditional

### A1. Semantic-Drift Benchmark + TimeSQL

Method novelty weakened by general bitemporal agent memory, but the evolving-business-semantics benchmark is still valuable.

### A2. CauseAlign-SQL

Potentially distinctive signal; needs a cheap proof-of-signal experiment.

### A3. DualSQL

Novel only if future-value probing is explicitly optimized and beats myopic exploration+cache.

### A4. EGraph-SQL

Clean efficiency/system contribution if safe equivalence classes compress ensemble redundancy enough.

---

## Tier B — keep in backlog

- IB-Warehouse
- GroupTest-Schema

## Killed / merged

- **SnorkelSQL** → killed by SQLENS direct prior art.
- JustificationGraph-SQL → merge into semantic drift/memory.
- LensSQL → merge into Result2SQL.

---

# 3. Stronger scientific thesis after the audit

The most promising new territory is no longer “how should an SQL agent reason harder?”

It is:

> **How does an agent turn experience and feedback into new database-level semantic structure?**

The top three candidates correspond to three distinct transformations:

1. **Feedback → repaired program** (`Result2SQL`)
2. **Solved programs → new abstractions** (`DreamSQL`)
3. **Repeated abstractions → executable infrastructure** (`AutoSemanticView`)

This suggests a deeper long-horizon ladder:

```text
raw interaction
   ↓
corrected SQL
   ↓
reusable semantic abstraction
   ↓
verified semantic asset
   ↓
cheaper + safer future queries
```

That ladder is more coherent—and more differentiated from current agent loops—than another planner/reviewer/controller stack.

---

# 4. Reviewer-facing claims we should avoid

Do **not** claim:

- first active schema exploration agent;
- first POMDP formulation of Text2SQL;
- first Text2SQL semantic memory;
- first semantic layer / semantic IR for NL2SQL;
- first use of why-not provenance for LLM query refinement;
- first weak-supervision SQL correctness model;
- first bitemporal memory for LLM agents;
- first query synthesis from input-output examples.

Those are contradicted or seriously weakened by nearby literature.

Safer claims should be task- and mechanism-specific and must be validated experimentally.

---

# 5. Newly added references from the adversarial search

- Park, Kim, Han. **SPARTA: Scalable and Principled Benchmark of Tree-Structured Multi-hop QA over Text and Tables.** ICLR 2026 / arXiv:2602.23286. Uses why-not provenance for LLM query refinement.
- Thakkar et al. **Example-Guided Synthesis of Relational Queries.** PLDI 2021, DOI:10.1145/3453483.3454098.
- Lee, Kim, Hwang. **Bootstrapping Semantic Layer from Execution for Text-to-SQL (GATE).** arXiv:2606.05634, 2026.
- Niksarli, Baheti. **A Graph-Native Bitemporal Memory Store for Conversational AI Agents.** arXiv:2607.26520, 2026.
- Wang. **TOKI: A Bitemporal Operator Algebra for Contradiction Resolution in LLM-Agent Persistent Memory.** arXiv:2606.06240, 2026.
- Xu. **Governed Persistent Memory: Source-Bound State Semantics and Fail-Closed Release for Long-Horizon Agents.** arXiv:2608.12476, 2026.
- **SQLENS: Fine-grained and Explainable Error Detection in Text-to-SQL.** OpenReview `CusEAujXDm`; weak-supervision aggregation of database + LLM SQL error signals.
