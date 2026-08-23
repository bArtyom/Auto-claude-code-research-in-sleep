# Round 2 — Falsifiable Experiment Plan

> **Role in ARIS-style workflow:** refined experiment plan after `research-refine`.  
> **Goal:** kill weak ideas cheaply before building a large agent system.  
> **Priority:** Pilot A = DreamSQL, Pilot B = Result2SQL. Pilot C = AutoSemanticView only if A finds useful abstractions.

# 0. General experimental rules

## 0.1 Use verified data first

Because recent work reports substantial annotation problems in common Text2SQL training data, the initial pilots should use either:

- a small expert-verified set of SQL programs; or
- controlled semantic mutants generated from verified programs.

Do not interpret disagreement with a noisy benchmark label as scientific failure without checking the SQL manually.

## 0.2 Matched-budget baselines

Every LLM comparison should match or report:

- model;
- token budget;
- number of model calls;
- database tool calls;
- wall-clock latency.

A new mechanism must not win only because it spends more inference.

## 0.3 Pre-register kill thresholds

The point of these pilots is decision-making, not demo creation. Thresholds below are intentionally concrete; revise them only before seeing results.

---

# Pilot A — DreamSQL: Is There a Learnable Semantic Library?

## A1. Scientific question

Does a repeated SQL workload contain reusable semantic subprograms whose explicit induction improves held-out compositional Text2SQL beyond trace/example retrieval?

## A2. Phase A0 — corpus feasibility scan

### Data

Target 300–1,000 verified queries from one or a small number of related database environments.

Candidate sources:

- one enterprise-like Spider 2.0 / dbt-style repository with many tasks;
- BIRD databases with enough within-schema queries;
- synthetic workload built from verified semantic motifs if benchmark density is insufficient;
- public analytics/dbt repos with historical analyst queries, if licensing permits.

### Preprocessing

Normalize SQL to a relational/semantic IR:

- canonical aliases;
- normalized commutative predicate order;
- explicit join graph;
- explicit grouping grain;
- canonical aggregate operators;
- time-expression normalization;
- optional logical entities/metrics if derivable from metadata.

### Feasibility metric

Run subtree/subgraph frequency analysis before any sophisticated library learner.

Record:

- fraction of programs containing a recurring nontrivial subprogram;
- support distribution of recurring subprograms;
- diversity of physical table names among structurally similar motifs;
- primitive-only description length.

### Gate A0

Proceed if at least one of the following is true:

- ≥20% of programs contain a recurring nontrivial motif appearing in ≥5 independent tasks; or
- structure-aware clustering reveals ≥10 motif families with clear cross-query reuse.

If not, DreamSQL may need a richer multi-task workload before it is worth pursuing.

---

## A3. Phase A1 — offline library induction

### Methods

1. **Frequency macro baseline** — extract most frequent normalized subtrees.
2. **MDL greedy baseline** — iteratively add abstraction maximizing description-length reduction.
3. **Stitch-style top-down library learner** — preferred first serious method.
4. Optional later: e-graph refactoring / DreamCoder-style wake-sleep.

### Output

A library:

```text
L = primitives ∪ {a1, a2, ... am}
```

Each abstraction records:

- formal parameters;
- body in semantic IR;
- support count;
- queries using it;
- compression gain;
- diversity of contexts;
- proposed human-readable description (non-authoritative).

### Metrics

- compression ratio;
- number of abstractions;
- average support;
- context diversity;
- percentage of corpus rewritten using abstractions;
- manual semantic-coherence score on top 20 abstractions.

### Gate A1

Proceed to generation only if:

- learned library improves corpus description length by ≥10% over primitive IR **and**
- at least 30% of top abstractions are judged semantically meaningful rather than formatting/syntax artifacts **and**
- useful abstractions occur across multiple distinct query intents.

Failure means representation/library objective needs revision before any agent experiment.

---

## A4. Phase A2 — motif-composition split

### Construction

For each program, represent which library motifs occur. Construct train/test splits that hold out **combinations**, not primitives.

Example:

```text
Train:
  active_customer + fiscal_period
  recognized_revenue + fiscal_period
  latest_snapshot + active_customer

Test:
  latest_snapshot + recognized_revenue
  active_customer + recognized_revenue + fiscal_period
```

Prevent near-duplicate SQLs from crossing the split using AST/IR similarity thresholds.

### Baselines

B0. one-shot model, no memory.

B1. top-k nearest SQL examples.

B2. top-k nearest semantic/IR examples.

B3. structured trace memory approximating AgentSM-style retrieval.

B4. learned library descriptions/examples inserted into context.

B5. learned library represented as callable semantic operators or constrained decoding primitives.

B6. library + retrieval.

### Evaluation

- execution correctness;
- expert semantic correctness on a reviewed subset;
- tokens;
- candidates sampled before success;
- library-operator usage rate;
- novelty of composition relative to training;
- error class.

### Primary hypothesis test

`B5/B6 > B2/B3` on held-out compositions at matched context budget.

### Gate A2 — kill threshold

DreamSQL is not supported if library-based methods improve semantic accuracy by <3 absolute points and do not reduce search/tokens by ≥15% relative to the strongest retrieval baseline.

A smaller accuracy gain may still be interesting only if token/search reduction is very large and robust.

---

## A5. Phase A3 — abstraction stress tests

### Test 1 — schema evolution

Rename/migrate physical tables while preserving semantic motif. Does the abstraction survive better than retrieved SQL examples?

### Test 2 — false abstraction

Inject a recurring syntactic pattern that has two different business meanings. Does the learner incorrectly merge them?

### Test 3 — negative transfer

Apply a learned abstraction to a task where one hidden assumption is violated. Can the system detect non-applicability?

### Test 4 — library size

Sweep number of abstractions and plot performance vs context/search cost.

### Desired artifact

A library quality report with examples of:

- successful compositional abstraction;
- syntactic junk abstraction;
- overgeneralized abstraction;
- undergeneralized abstraction.

---

# Pilot B — Result2SQL: Can One Corrected Result Item Repair the Program?

## B1. Scientific question

Does provenance-localized sparse denotational feedback repair executable semantic SQL errors more effectively and locally than strong LLM baselines given the same feedback?

## B2. Phase B0 — mutation benchmark

### Seed set

100–300 manually verified SQL queries across several databases.

### Mutation operators

Generate executable mutants from one semantic change:

- remove/add `DISTINCT`;
- `INNER ↔ LEFT JOIN`;
- wrong join key among plausible alternatives;
- wrong date column;
- shift date boundary;
- drop/add business predicate;
- swap `COUNT` / `COUNT DISTINCT`;
- `SUM ↔ AVG`;
- wrong grouping field;
- premature aggregation;
- wrong status/value mapping;
- missing bridge table.

### Filtering

Keep a mutant only if:

1. it parses and executes;
2. result differs from gold;
3. result is nontrivial (not simply empty due syntax-like failure);
4. error is understandable by a human reviewer;
5. one sparse output correction can provide useful evidence.

Target ≥500 mutant instances if generation is cheap; ≥150 is enough for the first pilot.

---

## B3. Feedback generation

From `gold_result` and `mutant_result`, sample one feedback item.

### Type N — unwanted tuple

Choose `r ∈ mutant \ gold`.

Feedback:

```text
This result row should not be present: <r>
```

### Type P — missing tuple

Choose `r ∈ gold \ mutant`.

Feedback:

```text
This known row should be present: <r>
```

### Type D — duplicate/equivalence

For dedup/grain errors, identify two rows that should collapse to one logical entity.

### Type A-lite — aggregate direction

If exact tuple difference is unavailable:

```text
The returned total is too high.
```

Exact corrected aggregate is a secondary condition because it assumes stronger user knowledge.

---

## B4. Repair methods

### R0 — regenerate

Question + schema + “previous answer is wrong”; regenerate.

### R1 — text-feedback baseline

Question + candidate SQL + feedback item.

### R2 — raw result-diff context

Add small relevant slices of mutant/gold-like feedback representation without provenance.

### R3 — clause perturbation search

Enumerate one-clause repairs from known mutation families; use feedback satisfaction to rank. This is a strong non-LLM baseline for controlled mutants.

### R4 — provenance-localized LLM repair

Compute why/why-not explanation slice and map it to implicated operators/clauses. LLM may edit only those regions initially.

### R5 — provenance-localized repair + collateral gate

R4 plus result preservation score on unflagged tuples/cells.

---

## B5. Practical provenance levels

Do not block the pilot on a complete provenance engine. Use three implementation levels.

### Level 1 — structural influence approximation

For each clause/operator, run controlled ablations or local rewrites and observe whether the flagged output changes. This gives an approximate influence map.

### Level 2 — database provenance library / lineage rewriting

Use available provenance/why-not tooling for supported relational fragment.

### Level 3 — causality/responsibility

Only after Level 1/2 show value, add responsibility ranking for ambiguous cases.

This staged plan tests the *value of localization* before investing in the full formal machinery.

---

## B6. Metrics

### Primary

- repaired SQL semantic correctness;
- execution match to verified gold;
- repair success after exactly one feedback item.

### Locality

- AST/IR edit distance from mutant;
- number of clauses changed;
- collateral output change:

```text
| outputs changed outside known error region |
```

### Efficiency

- tokens;
- DB executions;
- wall-clock;
- feedback items consumed before success.

### Breakdown

Report by mutation class and feedback type.

---

## B7. Gate B — kill threshold

The provenance/localization thesis is unsupported if R4/R5 does **not** improve one-feedback repair success by ≥5 absolute points over the strongest R1/R2 LLM baseline and does not reduce collateral damage.

If localization helps only one or two error families, narrow the paper to those families rather than generalizing.

---

# Pilot C — AutoSemanticView Offline Utility Simulation

> Run only after Pilot A demonstrates useful recurring abstractions, or independently using manually defined motifs for a quick systems sanity check.

## C1. Scientific question

Does promoting repeated semantic abstractions into executable maintained assets produce cumulative utility beyond semantic memory and classical execution-cost view selection?

## C2. Workload

Replay 500–2,000 queries with repeated motifs and controlled data/schema changes.

For each query record:

- semantic abstractions used;
- LLM tokens/calls;
- executed SQL plan/runtime/estimated cost;
- correctness;
- relevant schema version.

## C3. Candidate policies

P0. no asset creation.

P1. semantic memory only.

P2. frequency threshold: materialize when motif count > k.

P3. classical DB cost-only view selection.

P4. joint utility policy:

```text
reasoning savings
+ execution savings
- maintenance
- semantic risk
```

P5. oracle future workload.

## C4. Simulated assets

First implementation may represent an asset as a verified view or macro in DuckDB/Postgres. It does not need production dbt deployment.

## C5. Drift events

Inject:

- source schema change;
- metric definition change;
- workload shift;
- asset becomes unused;
- upstream data volume change.

Policies must decide whether to refresh, revise, or retire assets.

## C6. Metrics

Cumulative over workload:

- total LLM tokens;
- DB execution time/cost;
- maintenance work;
- semantic failures caused by stale assets;
- net utility vs no materialization;
- break-even query count for each asset.

## C7. Gate C

Continue only if P4 produces ≥15% cumulative utility improvement over both P1 and P3 under at least two realistic workload regimes and does not increase semantic error.

---

# Side Probe D — CauseAlign Signal Test

Before any dedicated architecture:

1. take 100 verified/mutant pairs;
2. create controlled tuple duplication/deletion interventions by entity class;
3. estimate each query's sensitivity signature;
4. compare correct vs mutant classification using:
   - AST features;
   - simple metamorphic tests;
   - sensitivity signature;
   - approximate responsibility ranking.

If causal/responsibility features add <2 AUROC points over cheap metamorphic features, do not pursue CauseAlign as a separate idea.

---

# Side Probe E — Candidate Equivalence Compression

Before building an e-graph:

1. collect multi-candidate SQL outputs from existing scaling/ensemble runs;
2. canonicalize aliases, whitespace, CTE layout, predicate order;
3. add a tiny sound rewrite set;
4. estimate how many candidates collapse into equivalence buckets.

Proceed to EGraph-SQL only if conservative transformations reduce verifier candidates by ≥20% without a detected false merge on a reviewed sample.

---

# Temporal Semantic Drift Benchmark — Initial Design

## T1. State model

Each semantic fact has:

```text
valid interval
knowledge interval
source / justification
entity / metric dependencies
```

## T2. Event types

- metric formula changes prospectively;
- metric is retrospectively restated;
- source table migration;
- join contract changes;
- source deprecation;
- agent learns late that an old assumption was wrong;
- conflicting sources overlap in time.

## T3. Query classes

1. current semantics/current data;
2. historical semantics/historical data;
3. current semantics applied retrospectively;
4. “what would the answer have been given what was known then?”

These classes force valid time and knowledge time to matter separately.

## T4. Baselines

- no memory;
- append-only memory;
- recency-weighted retrieval;
- temporal filtering;
- temporal + dependency invalidation.

## T5. Metrics

- stale semantic error;
- historical-answer accuracy;
- drift detection delay;
- unnecessary invalidations;
- error propagation to future tasks;
- recovery cost.

---

# Recommended execution order

```text
Day 1–2
├─ A0 corpus recurrence scan
└─ B0 semantic mutation benchmark prototype

Day 3–4
├─ A1 simple MDL/frequency abstraction mining
└─ B1/R1/R4 one-feedback repair comparison

Gate
├─ DreamSQL has meaningful reusable motifs? yes/no
└─ Result localization helps repair? yes/no

If YES:
Week 2
├─ A2 motif-composition evaluation
├─ B expanded mutation families + collateral metrics
└─ E equivalence-compression side probe

If DreamSQL survives:
Week 3+
└─ C AutoSemanticView offline simulator
```

# Decision table

| Pilot | Positive signal | Negative signal | Action |
|---|---|---|---|
| DreamSQL A0/A1 | nontrivial reusable semantic motifs + compression | only syntax/template repetition | continue / kill representation |
| DreamSQL A2 | compositional gain or major search reduction vs retrieval | retrieval matches library | kill/refocus as compression tool |
| Result2SQL | ≥5pt repair gain + lower collateral | raw feedback LLM matches | kill provenance mechanism or narrow feedback class |
| AutoSemanticView | ≥15% cumulative utility beyond memory + cost-only view selection | memory/cache captures gains | kill as standalone research |
| CauseAlign | responsibility adds signal beyond metamorphic tests | no incremental signal | kill |
| EGraph side probe | ≥20% safe candidate compression | little compression | kill |

# Final principle

Do not build the grand unified database agent yet.

The next step is empirical evidence for two small propositions:

1. repeated query solving contains **learnable semantic abstractions** that are more useful than retrieval;
2. sparse result correction contains **repair information that provenance/localization can exploit** better than a strong unconstrained LLM.

Those two results determine whether the larger long-horizon program has a scientific foundation.
