# Round 2 — Refined Research Proposal

> **Role in ARIS-style workflow:** `research-refine`  
> **Inputs:** literature landscape, candidate generation, novelty check, provisional critical review.  
> **Status:** research proposal; not cross-model accepted.  
> **Recommendation:** pursue two independent low-cost pilots first (`DreamSQL`, `Result2SQL`); keep `AutoSemanticView` as the systems follow-on.

# 0. Refined thesis

The literature strongly suggests that the next interesting Text2SQL contribution should not be another planner–critic loop.

The more defensible research object is:

> **How can a database agent transform interaction history into progressively more structured, reusable, and executable semantic knowledge?**

This creates a hierarchy:

```text
Question / task
   ↓
Generated SQL
   ↓
Observed result + sparse correction
   ↓
Verified repaired program
   ↓
Reusable semantic abstraction
   ↓
Executable semantic asset
   ↓
Cheaper and safer future interaction
```

The three proposed projects occupy distinct transitions in that hierarchy.

---

# 1. Flagship A — DreamSQL

## 1.1 Research question

> Can repeated successful query solving cause a Text2SQL agent to **invent a reusable semantic programming language** for one data environment, and does that language improve compositional generalization beyond example/trace retrieval?

## 1.2 Why this question exists after the literature check

Three nearby paradigms are already established:

1. **Trace/semantic memory:** AgentSM reuses prior structured programs.
2. **Execution-grounded memory:** GATE learns grounding hypotheses from execution and stores reusable results.
3. **Semantic layer compilation:** recent semantic-layer-mediated agents consume a high-level curated semantic model and compile it to physical SQL.

The missing transition is:

```text
solved workload
   → discover repeated semantic subprograms
   → invent new symbolic operators
   → rewrite old solutions with the operators
   → compose operators on genuinely new tasks
```

This is the DreamCoder / library-learning contribution translated to enterprise analytics.

## 1.3 Method sketch

### Representation

Normalize verified SQL into an intermediate semantic-relational representation. Avoid learning directly over raw SQL strings because dialect/syntax repetition can dominate abstraction discovery.

Possible primitive IR:

```text
Scan(entity/source)
Filter(predicate)
Join(relation, cardinality expectation)
TemporalFilter(time_semantics)
Dedup(key, ordering)
Aggregate(measure, grain)
Group(entity dimensions)
Rank(metric, k)
Project(outputs)
```

Add optional warehouse semantics when known:

```text
Entity<Customer>
Metric<RecognizedRevenue>
Grain<Customer, Month>
AsOf<date>
```

### Library learner

For the first pilot, use a fast abstraction miner inspired by **Stitch/top-down library learning**, not a full neural wake-sleep system.

Input:

- set of verified solved programs `P = {p_1...p_n}`;
- current primitive library `L_0`.

Output:

- abstractions `A = {a_1...a_m}` maximizing workload compression subject to minimum support and complexity constraints.

An abstraction is accepted only if it can replace subprograms in multiple independent tasks.

Example:

```text
latest_snapshot(entity, business_key, updated_at, as_of)
```

may compress many different queries whose surface SQL differs.

### Semantic naming

Naming should be downstream and non-authoritative. The LLM can propose names/descriptions for learned abstractions, but evaluation should depend on structure/reuse, not how pretty the names are.

### Retrieval vs library use

At test time compare:

- no memory;
- nearest solved examples;
- AgentSM-style program memory;
- learned library only;
- library + retrieval.

The library is successful only when it helps on **novel compositions**, not just nearest neighbors.

## 1.4 Key benchmark construction

Standard random splits are inadequate because train/test tasks can share entire templates.

Create a **motif-composition split**:

- detect or define semantic motifs `A`, `B`, `C`;
- training contains tasks with `A`, `B`, `C`, `A+B`, `B+C`;
- test contains held-out combination `A+C` or `A+B+C`;
- prevent near-duplicate SQL leakage.

Possible motifs:

- latest snapshot;
- entity deduplication;
- active status as-of date;
- recognized revenue;
- fiscal period transformation;
- cohort membership;
- slowly changing dimension join.

This directly tests whether the induced library behaves like a language rather than a cache.

## 1.5 Hypotheses

### H1 — compression

A learned library reduces description length of a verified SQL/IR workload compared with primitive-only representation.

### H2 — compositional transfer

At fixed context/token budget, library-augmented generation improves execution/semantic correctness on held-out motif compositions relative to example/trace retrieval.

### H3 — search efficiency

The library reduces generation/search tokens or candidate count required to reach a correct query.

### H4 — abstraction quality is predictive

Abstractions with high corpus compression + diverse task support transfer better than abstractions selected only by frequency.

## 1.6 Main falsifiers

Kill or heavily reformulate DreamSQL if:

- learned abstractions mostly encode physical table/column names and do not transfer compositionally;
- retrieval memory matches performance at equal persistent-state budget;
- compression improves but semantic accuracy/search does not;
- useful abstractions require so much manual semantic annotation that automatic induction is not the actual contribution.

## 1.7 Potential paper contribution

A clean paper can contribute:

1. a formal Text2SQL **semantic library induction** task;
2. a workload-to-library algorithm;
3. a motif-composition evaluation protocol;
4. evidence that induced operators improve long-horizon query solving beyond memory retrieval.

### Working title

**DreamSQL: Learning the Semantic Language of a Data Warehouse from Repeated Query Solving**

---

# 2. Flagship B — Result2SQL

## 2.1 Research question

> Can sparse corrections on an observed query result serve as a more efficient supervision channel for repairing executable semantic SQL errors than textual clarification or unconstrained regeneration?

## 2.2 Task definition

Given:

- natural-language intent `q`;
- database `D`;
- generated candidate SQL `s`;
- result `R = s(D)`;
- one sparse user correction `f` on `R`;

produce repaired SQL `s'` satisfying the correction while preserving as much of the previously correct behavior and the original NL intent as possible.

Feedback types:

### Negative tuple

```text
“This row should not be present.”
```

### Positive tuple

```text
“This known entity should appear.”
```

### Equivalence / duplicate correction

```text
“These two result rows represent the same customer.”
```

### Aggregate directional feedback

```text
“This total is too high.”
```

### Optional exact cell correction

```text
“This value should be 14, not 17.”
```

The first paper should prioritize negative/positive tuple feedback because it is realistic and clean.

## 2.3 Method sketch

### Step A — provenance/localization

Compute an explanation slice for the corrected output:

- why-provenance for unwanted tuples;
- why-not provenance or candidate missing-support explanations for absent tuples;
- approximate operator influence for aggregate feedback.

### Step B — candidate semantic fault localization

Map the provenance slice to candidate clauses/operators likely responsible:

- join condition/type;
- filter predicate;
- aggregation/distinctness;
- grouping grain;
- temporal condition;
- entity mapping.

### Step C — constrained repair

Generate small semantic edits only in the localized region. Score repaired programs on:

```text
feedback satisfaction
+ intent consistency
+ preservation of unflagged outputs
+ minimal semantic edit
+ execution validity
```

### Step D — collateral-damage check

Re-execute and reject repairs that satisfy the corrected row by breaking many previously correct rows.

## 2.4 Novelty discipline

The paper must explicitly acknowledge:

- why/why-not provenance literature;
- query causality;
- example-guided relational query synthesis;
- relational lenses;
- SPARTA's provenance-driven LLM query refinement.

The contribution is **not provenance itself**.

The contribution is a new interactive Text2SQL problem/interface plus a provenance-constrained minimal-repair algorithm and evaluation.

## 2.5 Benchmark generation

A controlled benchmark can be created from verified SQL:

1. execute gold SQL;
2. produce an executable semantic mutant;
3. execute mutant;
4. compute output difference;
5. select one sparse correction that distinguishes mutant from gold;
6. hide gold SQL from the repair system.

Mutation families:

- `INNER ↔ LEFT JOIN`;
- remove/add `DISTINCT`;
- wrong grouping field;
- wrong date column;
- off-by-one boundary;
- dropped predicate;
- swapped business status;
- wrong aggregation (`COUNT` vs `COUNT DISTINCT`, `SUM` vs `AVG`);
- wrong bridge table;
- premature aggregation.

Only keep mutants that execute and produce plausible nontrivial results.

## 2.6 Hypotheses

### H1 — sparse feedback is high bandwidth

One corrected tuple yields higher repair success than a generic textual “the answer is wrong” message and approaches a detailed natural-language bug report.

### H2 — provenance improves locality

Provenance-guided repair changes fewer unrelated clauses and causes less collateral result damage than raw-result-diff repair.

### H3 — multi-turn convergence

When one correction is insufficient, the system can request/consume a second correction and converge faster than regenerate-and-review baselines.

## 2.7 Main falsifiers

Kill or narrow the project if:

- a capable LLM using raw result diff matches provenance-guided repair;
- provenance slices are too large/unstable to localize common semantic bugs;
- useful feedback requires users to know the exact gold result;
- repaired SQL overfits the marked tuple and damages the rest of the result.

## 2.8 Potential contribution

1. **denotational-feedback Text2SQL repair** task;
2. automatically generated benchmark of sparse output corrections;
3. provenance-constrained minimal repair;
4. feedback-efficiency and collateral-damage metrics.

### Working title

**Result2SQL: Sparse Denotational Feedback for Interactive Text-to-SQL Repair**

---

# 3. Systems follow-on — AutoSemanticView

## 3.1 Research question

> When should a long-lived data agent promote repeated semantic reasoning into an executable, maintained semantic asset?

## 3.2 Why this should come after DreamSQL

DreamSQL answers:

> Which reusable abstractions exist?

AutoSemanticView answers:

> Which of those abstractions are worth turning into infrastructure?

The second decision requires workload statistics, query cost, maintenance burden, semantic confidence, and drift risk.

## 3.3 Asset lifecycle

```text
repeated verified program fragment
       ↓
candidate abstraction
       ↓
frequency / benefit estimate
       ↓
semantic confidence gate
       ↓
materialize as:
  view / dbt model / macro / metric
       ↓
monitor use + maintenance
       ↓
refresh / revise / merge / retire
```

## 3.4 Objective

For asset `v` over horizon `T`:

```text
U(v) =
  Σ expected LLM reasoning saved
+ Σ expected DB execution saved
+ Σ expected semantic-risk reduction
- creation cost
- maintenance cost
- storage cost
- drift-induced risk
```

A semantic asset should not be materialized merely because it is frequent. High uncertainty or rapid semantic drift should block promotion.

## 3.5 Critical baselines

- semantic memory only;
- static curated semantic layer;
- classical materialized-view selection on execution cost only;
- frequency threshold;
- oracle future workload.

## 3.6 First research artifact

Build a workload replay simulator rather than immediately writing to production dbt repositories.

### Working title

**AutoSemanticView: When Should a Data Agent Turn Reasoning into Infrastructure?**

---

# 4. Supporting benchmark — Temporal Semantic Drift

Rather than a standalone “bitemporal agent memory” method, create an evolving-warehouse benchmark with two clocks:

- when a semantic rule is valid;
- when the agent learns/corrects it.

Scenario classes:

1. metric redefinition;
2. canonical source migration;
3. fiscal calendar change;
4. SCD/join rule change;
5. late-arriving correction to historical semantics;
6. source deprecation;
7. contradictory authority sources;
8. retrospective restatement.

Questions include both current and historical intent:

```text
“What is ARR now?”
“What was ARR under the definition used in Q2 2024?”
“Recompute Q2 2024 using the definition adopted in 2026.”
```

This benchmark can later test DreamSQL/AutoSemanticView invalidation and repair.

---

# 5. Research program map

The proposals can eventually form one coherent program without forcing them into one paper:

```text
                    ┌──────────────────────┐
                    │ Sparse user feedback │
                    └──────────┬───────────┘
                               │
                               ▼
                     Result2SQL repair
                               │
                         verified programs
                               │
                               ▼
                     DreamSQL abstraction
                               │
                    reusable semantic DSL
                               │
                               ▼
                   AutoSemanticView policy
                               │
                maintained semantic assets
                               │
                               ▼
           lower future cost + fewer silent errors

Temporal semantic drift benchmark stresses all three stages.
```

This produces a more original long-horizon perspective:

> **A mature database agent should not merely answer more questions. It should gradually transform repeated experience into a safer semantic interface to the data world.**

---

# 6. Paper sequencing recommendation

## Paper 1A — lowest-risk empirical paper

**Result2SQL**

Why first:

- benchmark can be synthesized from verified SQL;
- pilot is cheap;
- success/failure is easy to measure;
- no model training required initially;
- strong bridge between DB theory and LLM interaction.

## Paper 1B — highest-upside conceptual paper

**DreamSQL**

Why in parallel:

- if reusable semantic abstractions emerge, the idea has a larger long-horizon thesis;
- first pilot can be offline library mining before training any agent.

## Paper 2 — systems extension

**AutoSemanticView**

Depends on evidence that useful reusable abstractions actually exist.

## Benchmark / shared infrastructure

**Temporal Semantic Drift** can be developed incrementally and later become a standalone benchmark paper if it reveals clear failures in current memory systems.

---

# 7. What not to build yet

Do not start with:

- a huge multi-agent controller;
- RL over database tools;
- a full production dbt writer;
- a new foundation Text2SQL model;
- a general bitemporal memory server;
- a complex causal-responsibility engine;
- an e-graph optimizer for every SQL dialect.

The next scientific action is to answer two cheap binary questions:

1. **DreamSQL:** Does a real repeated SQL workload contain abstractions that improve *held-out compositional solving* beyond retrieval?
2. **Result2SQL:** Does provenance-localized sparse result feedback improve *repair success and locality* beyond a strong LLM seeing the same result difference?

If either answer is “no,” we should kill or substantially reformulate the corresponding idea before architecture work.

---

# 8. Current proposal status

| Proposal | Novelty after audit | Cheap falsification | Engineering risk | Recommendation |
|---|---:|---:|---:|---|
| DreamSQL | high | good | medium | run pilot now |
| Result2SQL | medium-high/high after narrow framing | excellent | low-medium | run pilot now |
| AutoSemanticView | medium-high | good via simulator | high | follow-on |
| Temporal Semantic Drift benchmark | medium-high benchmark novelty | good | medium | develop shared infra |
| CauseAlign-SQL | uncertain | excellent signal probe | medium | probe only |
| DualSQL | medium | good | medium-high | backlog |
| EGraph-SQL | medium-high efficiency novelty | excellent | medium | cheap side experiment |

**Final recommendation:** commit resources first to the DreamSQL and Result2SQL pilots. They test two different, literature-grounded hypotheses and fail cheaply.
