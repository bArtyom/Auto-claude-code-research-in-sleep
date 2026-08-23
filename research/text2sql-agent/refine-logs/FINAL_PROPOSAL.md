# Final Proposal — SemLibSQL-Γ

**Paper-facing title:** **SemLibSQL: Learning Reusable SQL Abstractions with Warehouse-Conditioned Equivalence**  
**Date:** 2026-08-23  
**Verdict:** **READY FOR FALSIFICATION EXPERIMENT; NOT EMPIRICALLY VALIDATED**

---

## 1. Problem Anchor

Long-lived Text2SQL systems increasingly reuse experience through verified-query repositories, structured execution memory, semantic grounding memory, and curated semantic layers. Generic program-synthesis research also shows how to learn reusable functions from corpora of solved programs.

The unresolved problem is more specific to relational data systems:

> **Useful SQL reuse is often conditional on local warehouse laws.**

Two implementations may be interchangeable only if a key is unique, a relationship is total, timestamps obey a particular validity policy, a bridge table is lossless, or a business definition is governed by a specific contract. Conversely, two visually similar SQL fragments may differ critically because one such condition does not hold.

A memory system stores previous solutions. A generic library learner compresses repeated program structure. A semantic layer provides manually or automatically constructed business concepts. None of these by itself tests whether **warehouse-conditioned equivalence should be the organizing principle for learning a reusable reasoning language**.

---

## 2. Final Method Thesis

> **Learn reusable Text2SQL operators together with the warehouse conditions that make their supporting implementations equivalent.**

SemLibSQL-Γ induces a library of parameterized relational operators from verified SQL histories. It differs from generic library learning by treating equivalence as **conditional on a warehouse contract** and by attaching the relevant applicability contract to every induced operator.

A learned item is therefore not just:

```text
latest_snapshot(relation, key, time, as_of)
```

but:

```text
operator: latest_snapshot(relation, key, time, as_of)
requires:
  key is unique at the target grain
  time defines the intended validity ordering
  as_of excludes future records
  tie behavior is defined
supported_by:
  [verified instances / proof or counterexample evidence]
```

The operator can be reused only when the current context satisfies its scope contract.

---

## 3. Dominant Contribution

**Contract-conditioned semantic library induction for Text2SQL.**

The paper should make exactly one headline claim:

> A library learner that reasons over SQL equivalence **under warehouse-specific contracts** discovers safer/more reusable abstractions than strong syntax/global-theory refactoring baselines and yields compositional reuse beyond episodic memory.

Everything else—LLMs, e-graphs, SMT/equivalence checking, execution, retrieval—is supporting machinery.

---

## 4. Representation

### 4.1 Verified experience

Each training/library item contains:

```text
natural-language request
warehouse/schema/repository context
verified SQL
execution evidence
available metadata/contracts
```

Only verified experiences can provide positive support for an operator. Failed queries may generate hard negatives but cannot directly define library semantics.

### 4.2 Typed relational plan

Lower SQL into a representation exposing at least:

- scans / logical sources;
- joins and join predicates;
- filters;
- grouping/output grain;
- aggregations and distinctness;
- temporal predicates/window functions;
- projection/lineage;
- key and null semantics where known.

Add domain-level types only where they affect reuse safety, e.g.:

```text
Entity<Customer>
Grain<Customer, Month>
Snapshot<Customer, ValidTime>
Measure<RecognizedRevenue>
Key<CustomerID>
```

Avoid a large ontology in the first experiment.

---

## 5. Warehouse Contract Γ

`Γ` is a set of scoped facts with provenance/confidence.

### 5.1 Preferred evidence sources

1. declared primary/foreign keys;
2. declared uniqueness/nullability;
3. dbt tests/contracts;
4. version-controlled semantic-layer definitions;
5. validated lineage rules;
6. repeated data-profile invariants, marked as empirical rather than guaranteed;
7. business rules that have explicit supporting evidence.

### 5.2 Evidence classes

Every fact should be typed as one of:

- `DECLARED` — explicit schema/contract fact;
- `VERIFIED` — supported by a strong check/proof/test;
- `EMPIRICAL` — observed in data, not guaranteed globally;
- `HYPOTHESIS` — not safe for automatic abstraction reuse.

Initial Gate A should prioritize `DECLARED` + `VERIFIED`; otherwise failures may be impossible to diagnose.

---

## 6. Contextual Equivalence Oracle

For programs/subplans `P` and `Q`, SemLibSQL seeks evidence for:

`P ≡_Γ Q`.

The oracle is intentionally incomplete.

### 6.1 Three-valued result

```text
EQUIVALENT
NOT_EQUIVALENT
UNKNOWN
```

`UNKNOWN` is a valid terminal judgment. High precision matters more than aggressive merging.

### 6.2 Evidence stack

Use the strongest cheap evidence available:

1. deterministic safe rewrites / canonical relational normalization;
2. formal or bounded equivalence checking under integrity constraints for the supported SQL subset;
3. contract-preserving differential execution on generated/adversarial database instances;
4. lineage/grain/invariant comparisons;
5. explicit counterexamples immediately force `NOT_EQUIVALENT`.

LLM judgments can propose candidates or explanations but should not independently certify equivalence.

---

## 7. Abstraction Induction

### 7.1 Candidate generation

Start from repeated or related subplans proposed by:

- AST/plan similarity;
- anti-unification;
- shared output type/grain;
- shared NL/business concept;
- workload co-occurrence.

Candidate generation can be permissive because the equivalence/scope stage is conservative.

### 7.2 Learn abstraction and scope jointly

For a candidate family, infer:

```text
operator body / semantic skeleton
parameters
supporting instances
sufficient applicability conditions
known counterexamples
```

The first paper does **not** need the mathematically weakest precondition. A compact, evidence-backed sufficient scope is enough.

### 7.3 Objective

A candidate operator is valuable when it improves a tradeoff such as:

```text
reuse/compression benefit
+ expected search reduction
+ cross-realization coverage
+ downstream composition utility
- parameter complexity
- scope complexity
- unsupported merges
- negative transfer risk
```

Do not optimize compression alone; babble/Stitch are already strong at that objective.

---

## 8. Library-Aware Text2SQL

For a new question:

```text
question + warehouse context
        ↓
identify relevant learned operators
        ↓
check operator scope conditions
        ↓
compose in-scope operators + residual relational logic
        ↓
lower/realize SQL
        ↓
normal execution/validation
```

If no operator is safely applicable, use the base Text2SQL method. Library use is selective, not mandatory.

---

## 9. Example of Contextual Equivalence

Suppose two historical queries compute one row per customer at an `as_of` date.

Implementation A:

```text
ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY effective_at DESC)
```

Implementation B:

```text
MAX(effective_at) GROUP BY customer_id
JOIN back to source
```

They are not automatically equivalent in all databases. Equivalence may require a condition such as:

```text
(customer_id, effective_at) is unique
```

or an explicit tie-resolution rule.

A generic syntactic learner sees different structures. A generic equality theory may merge them only if that rule is supplied globally. SemLibSQL-Γ instead stores the learned motif together with its local applicability contract.

This kind of conditional reuse is the specific phenomenon the experiment must validate.

---

## 10. Claims

### C1 — Mechanism claim

Warehouse-conditioned equivalence improves the precision–recall frontier of reusable abstraction discovery versus strong generic library-learning/refactoring baselines.

### C2 — Downstream claim

Scoped learned operators improve held-out motif compositions over verified-query/structured-memory baselines at matched context budget.

### C3 — Safety claim

Applicability contracts materially reduce false reuse / negative transfer versus the same learned abstractions without scope checks.

---

## 11. Required Baselines

### Program-library baselines

- Stitch / syntactic library learning;
- babble/LLMT with a strong fixed SQL equational theory;
- ReGAL-style execution-validated refactoring.

### SQL-equivalence components/baselines

- safe logical-plan normalization;
- constraint-aware equivalence checker where practical;
- execution/differential tests.

### Text2SQL reuse baselines

- nearest verified query retrieval;
- matched-token maximal retrieval;
- AgentSM-style structured program memory;
- execution-grounded semantic memory;
- manual/oracle semantic operator library.

---

## 12. Explicit Non-Claims

Do not claim:

- first program-library learning;
- first library learning modulo equivalence;
- first SQL equivalence checker;
- first equivalence-under-integrity-constraints method;
- first SQL workload mining;
- first Text2SQL memory;
- first semantic-layer induction;
- first compositional Text2SQL evaluation;
- universal equivalence across databases/dialects;
- full automatic business-ontology discovery.

---

## 13. Key Risks and Responses

### Risk A — “babble already solves this.”

**Response required by experiment, not rhetoric:** give babble/LLMT a strong fixed SQL theory and show that warehouse-specific contextual conditions recover additional safe abstractions or downstream composition benefit.

### Risk B — “This is old semantic query optimization.”

Constraint-aware equivalence is old. The contribution must be abstraction/language induction and Text2SQL compositional reuse, not query optimization.

### Risk C — “Semantic View Autopilot already learns business semantics.”

Avoid broad semantic-layer claims. The experiment object is scoped program abstraction versus PL/memory baselines, not semantic-model authoring productivity.

### Risk D — “You hand-coded the semantics.”

Report exactly which contract facts are declared, inferred, and manual. If success requires hand-labeling each motif, the automatic-induction claim fails.

### Risk E — unsafe overgeneralization

Hard negatives and scope-violation tests are mandatory. An abstraction with high recall but unacceptable false merges is a failed abstraction.

---

## 14. Complexity Intentionally Rejected

For the first paper, do **not** add:

- RL fine-tuning;
- MCTS;
- multi-agent debate;
- general-purpose long-term memory architecture;
- automatic semantic-view materialization;
- active schema POMDP;
- large ontology induction;
- cross-user governance workflows.

They obscure the only important question: **does contract-conditioned equivalence change useful library induction?**

---

## 15. Paper Narrative if Gate A/B Pass

1. Repeated Text2SQL experience contains latent reusable programs.
2. Generic library learning handles syntactic variation, but SQL reuse is often conditional on local warehouse laws.
3. SemLibSQL-Γ learns scoped operators under warehouse-conditioned equivalence.
4. Controlled hard-positive/hard-negative experiments show the mechanism changes the abstraction frontier.
5. Held-out composition shows the learned language adds value beyond verified memory.
6. Scope contracts prevent the main failure mode: semantic overgeneralization.

---

## 16. Stop Condition

This proposal is now specific enough that further idea-stage discussion cannot establish its central claims. The next step requires **real SQL corpus construction and execution**.

No pilot result is claimed here.
