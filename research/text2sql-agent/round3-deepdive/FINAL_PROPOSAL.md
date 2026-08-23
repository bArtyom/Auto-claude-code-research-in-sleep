# Final Proposal — SemLibSQL

> **Project codename:** DreamSQL  
> **Paper-facing name:** **SemLibSQL: Learning a Warehouse-Specific Semantic Library from Verified Text-to-SQL Experience**  
> **Date:** 2026-08-23  
> **Workflow role:** `research-refine` output  
> **Current verdict:** **READY FOR CHEAP PILOT**  
> **Formal cross-model acceptance:** unavailable in this connector session; status remains provisional.

## 1. Problem anchor

Modern Text2SQL agents can use retrieval, execution, clarification, multi-candidate search, and persistent memory. Recent systems also show that verified prior queries or structured agent traces can improve future tasks in the same database.

But these approaches mostly make the agent's **memory larger**. They do not test whether experience can make the agent's **reasoning language better**.

In a repeated enterprise workload, many SQL solutions instantiate the same hidden business operation in different syntactic forms:

- latest valid snapshot;
- active-customer state;
- fiscal-period mapping;
- deduplicate-before-join;
- recognized-revenue definition;
- bridge-table traversal;
- slowly changing dimension lookup;
- eligible-population denominator.

A human analyst gradually stops re-deriving these from raw tables. They acquire reusable concepts.

The research question is:

> **Can a Text2SQL agent automatically induce its own warehouse-specific semantic operators from verified experience, and do those operators enable compositional generalization beyond retrieving old queries?**

---

## 2. Dominant contribution

SemLibSQL introduces **semantics-aware program-library induction for Text2SQL**.

Given many verified `(question, context, SQL)` experiences from one warehouse, it:

1. parses and grounds SQL into a typed relational/semantic plan;
2. canonicalizes implementations that are behaviorally equivalent despite different SQL syntax/dialects/decompositions;
3. mines repeated parameterized substructures across the canonicalized corpus;
4. promotes high-purity candidates into typed semantic operators only after evidence/behavior validation;
5. exposes the learned operators as a compact program library for future Text2SQL generation;
6. evaluates whether that learned language solves **held-out compositions** better and more cheaply than verified-query memory.

The paper is **not** primarily about memory, semantic layers, agent skills, or SQL compilation. Those are neighboring literatures/baselines.

---

## 3. Method

### 3.1 Experience store

Each accepted experience contains:

```text
natural-language request
schema / repository context used
verified SQL
execution evidence
(optional) semantic metadata: entity, grain, metric, time context
```

Only verified experiences enter library induction. Failed or uncertain traces can be used as negatives but must not define primitives.

### 3.2 Typed semantic normalization

Convert SQL into a representation that exposes at least:

- source entities/tables;
- joins and relationship cardinalities where known;
- filters;
- grouping/output grain;
- aggregations;
- temporal operators;
- window operators;
- projections;
- key/distinctness semantics;
- lineage to output fields.

Types may include domain-level annotations such as:

```text
Entity<Customer>
Grain<Customer, Month>
Money<USD, Recognized>
Snapshot<Entity, Time>
Metric<Revenue>
```

The type system should be minimal: include only what improves canonicalization or safe abstraction reuse.

### 3.3 Semantic canonicalization

This is the core SQL-specific mechanism.

Use a conservative combination of:

- relational rewrite rules;
- e-graph/equality-saturation style equivalence where safe;
- schema-aware type/grain normalization;
- behavior signatures from execution on discriminating database instances or perturbations;
- lineage/provenance signatures;
- invariant checks.

Goal: align implementations of the same semantic motif that AST compression would treat as different.

Do **not** claim complete SQL equivalence checking. The system may maintain conservative equivalence clusters with an `unknown` state.

### 3.4 Corpus-level abstraction induction

Mine repeated parameterized structures across canonical plans.

A candidate primitive should optimize a tradeoff such as:

```text
compression/reuse benefit
+ downstream search reduction
+ semantic consistency
- parameter complexity
- ambiguity / context dependence
- validation failures
```

Generic library-learning algorithms such as Stitch/DreamCoder-style compression can be used as baselines or components, but the abstraction search operates over the semantic representation rather than raw SQL syntax.

### 3.5 Primitive validation

A candidate becomes a reusable operator only if it passes tests such as:

- instances share compatible types/grain;
- parameter substitution behaves consistently;
- execution/invariant tests agree;
- no known counterexample separates supposedly equivalent instances;
- support comes from multiple independent experiences, not one template duplicate.

Each operator stores a scope/evidence record so the generator can choose not to use it outside its valid context.

### 3.6 Library-aware generation

For a new question:

```text
question + warehouse context
      |
      v
semantic sketch / retrieval of relevant operators
      |
      v
compose primitive calls + residual relational operations
      |
      v
lower to SQL / let base model realize SQL
      |
      v
normal validation/execution
```

The library is optional. If no primitive fits, the system falls back to the base Text2SQL procedure.

---

## 4. Example

Suppose historical queries independently implement:

```text
A. latest customer record as of month end
B. latest subscription record as of a reporting date
C. latest account state before event time
```

Their SQL surfaces differ dramatically. After semantic normalization, they share:

```text
argmax-per-key under time <= as_of
```

SemLibSQL can induce:

```text
latest_snapshot(
    relation,
    business_key,
    effective_time,
    as_of
)
```

Later, a question requiring:

```text
active_customer(as_of)
+
latest_snapshot(...)
+
recognized_revenue(period)
```

may be a combination never seen as a full SQL query. The experiment asks whether composition from induced primitives succeeds where nearest-query memory does not.

---

## 5. Claims

### Main claim

**C1 — Compositional generalization beyond episodic memory.**

Automatically induced semantic operators improve accuracy and/or reasoning efficiency on held-out combinations of known warehouse motifs compared with strong verified-query retrieval and structured trace memory.

### Mechanism claim

**C2 — Semantic canonicalization matters.**

Canonicalizing behaviorally equivalent SQL before library mining yields higher-purity/more reusable operators and better downstream composition than syntactic AST compression alone.

### Representation claim

**C3 — Learned operators capture stable semantics.**

Induced primitives exhibit consistent type/grain/behavior across instances and controlled interventions, rather than merely compressing surface syntax.

### Long-horizon claim

**C4 — Experience changes the agent's language and reduces future reasoning cost.**

As verified experience accumulates, future queries that reuse learned motifs require fewer tokens/tool steps without increased silent error.

C4 is secondary; do not oversell a continual-learning story before C1–C3 succeed.

---

## 6. What is deliberately excluded

To keep the paper focused, do **not** include as central components:

- result-table feedback / DenoRepair;
- automatic semantic-view/materialized-view creation;
- long-term belief revision or bitemporal memory;
- active schema exploration;
- multi-agent debate;
- proof-carrying SQL;
- RL fine-tuning;
- autonomous data-governance workflows.

If the core result is positive, these can be follow-up projects.

---

## 7. Closest-prior positioning

### Versus AgentSM / verified-query memory

They retain/retrieve prior experiences. SemLibSQL **induces a new parameterized program vocabulary** from many experiences and evaluates unseen compositions.

### Versus GATE / semantic grounding memory

GATE learns validated grounding knowledge. SemLibSQL targets **higher-order reusable relational/business operators** that compose into new programs.

### Versus DreamCoder / Stitch / LILO

They establish general library learning. SemLibSQL must contribute the **SQL-semantic canonicalization and validation problem** and demonstrate why it changes abstraction discovery and Text2SQL composition.

### Versus generic agent skill libraries

Those learn procedures/tool macros across agent tasks. SemLibSQL focuses on **database-semantic operators under SQL equivalence, grain, entity, and business-context constraints**.

### Versus semantic-layer agents

Those consume a curated or separately bootstrapped semantic layer. SemLibSQL asks whether the layer's useful operators can be **automatically induced from verified query experience**.

---

## 8. Success criteria

The idea deserves full implementation only if the pilot shows all of:

1. semantic canonicalization aligns important motifs missed by syntactic mining;
2. induced operators have high semantic purity on manual/intervention audit;
3. held-out composition accuracy improves beyond strong query memory;
4. gains survive matched context/token budgets;
5. negative transfer is small and optional routing mitigates it;
6. at least one enterprise-like/open real workload shows the effect.

---

## 9. Kill criteria

Stop or fundamentally reframe if:

- semantic canonicalization does not improve over AST/library compression;
- retrieval of verified SQL performs equally well on held-out composition;
- induced abstractions mostly reproduce SQL syntax/CTEs;
- automatic induction adds no value over a small manually written semantic library;
- gains require synthetic tasks but disappear on real workloads;
- abstraction reuse introduces substantial silent errors.

Do not rescue a failed pilot by adding more agents, larger models, or RL.

---

## 10. Paper story in one paragraph

Text2SQL systems increasingly remember verified solutions, but memory does not necessarily create reusable concepts. We ask whether repeated database experience can instead change an agent's program language. SemLibSQL semantically normalizes verified SQL programs, induces a compact typed library of recurring warehouse operations, validates the learned operators, and reuses them compositionally. On leakage-controlled motif-composition splits and enterprise-like workloads, we compare against stateless generation, query retrieval, structured semantic memory, generic syntactic library learning, and manually authored semantic primitives. The central test is whether learned semantic operators—not stored answers—enable new combinations with lower inference cost and low negative transfer.

---

## 11. Current status

```yaml
method_thesis: stable_for_pilot
novelty: conditional_medium_high
external_review: unavailable
formal_acceptance: provisional
next_stage: cheap_anchor_pilot
```

The next artifact, `EXPERIMENT_PLAN.md`, intentionally begins with the smallest experiment capable of killing the thesis.