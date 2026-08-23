# Final Proposal — SemLibSQL-Γ

**Working title:** **SemLibSQL-Γ: Learning Guarded Semantic Operators from Verified Text-to-SQL Workloads**  
**Date:** 2026-08-23  
**Verdict:** **READY FOR GATE-A FALSIFICATION; NOT EMPIRICALLY VALIDATED**

---

## 1. Problem anchor

Long-lived Text2SQL agents increasingly reuse prior experience through verified-query retrieval, structured semantic memory, execution-grounded memory, and semantic layers. At the same time, program-synthesis research already learns reusable libraries from program corpora, including library learning over e-graphs/equational theories.

The remaining SQL-specific research question is narrower:

> **Does enterprise SQL contain a meaningful Guarded Abstraction Gap—recurring semantic operations that are safely reusable only after conditioning equivalence on local warehouse contracts—and can those guarded operations improve new Text2SQL compositions beyond memory and generic program-library learning?**

The project is therefore about a property of the workload first and an algorithm second.

---

## 2. Why the claim had to be narrowed

The following mechanisms are explicitly **not new**:

- reusable program-library induction: DreamCoder, Stitch;
- library learning modulo an equational theory: babble/LLMT;
- library learning directly over e-graphs: E-Stitch (EGRAPHS 2026);
- execution-validated abstraction/refactoring: ReGAL;
- equality under assumptions/conditional rewriting: conditional rewriting, Colored E-Graphs, Predicate E-Graphs (2026);
- property analysis to trigger conditional rewrites: e-graphs + abstract interpretation;
- automatic rule precondition inference: Alive-Infer and broader precondition-inference literature;
- SQL equivalence/reformulation under integrity constraints: mature database literature, including recent bounded/formal tools;
- reusable Text2SQL semantic memory: AgentSM, GATE and related systems.

Consequently, the paper cannot be “e-graphs for SQL library learning” or “learn preconditions for reusable SQL functions.”

---

## 3. Dominant contribution

### Scientific claim

Measure and exploit the **Guarded Abstraction Gap (GAG)**:

> the additional reusable semantic structure revealed when equivalence is conditioned on local warehouse laws rather than syntax or a globally supplied/fixed equational theory, while holding false semantic merges low.

### Method claim, only if experiments support it

Jointly induce:

```text
(operator, applicability guard, evidence)
```

from verified Text2SQL histories, and optimize the library for downstream compositional reuse rather than corpus compression alone.

If a pipeline built from E-Stitch/babble + predicate/conditional equality + Alive-Infer-style guard inference matches the joint method, the method claim must be dropped. A benchmark/analysis contribution is still possible only if the Guarded Abstraction Gap itself is large and interesting.

---

## 4. Formal object

Let `Γ` be a warehouse contract composed of scoped facts such as:

- primary/foreign keys;
- uniqueness;
- nullability;
- relationship coverage/cardinality;
- dbt tests/contracts;
- validated lineage;
- governed metric/business definitions;
- evidence-backed invariants.

For SQL plans/subplans `P` and `Q`, seek contextual equivalence:

`P ≡_Γ Q`.

This does not mean global SQL equivalence. It means the relevant observable behavior is equivalent when the required portion of `Γ` holds.

### Contract provenance

Every fact is one of:

```text
DECLARED
VERIFIED
EMPIRICAL
HYPOTHESIS
```

The first experiment permits only `DECLARED` + `VERIFIED` facts to certify automatic reuse; empirical/hypothesized facts are ablations or candidate-discovery signals.

---

## 5. Guarded operator representation

Each learned primitive is an explicit contract-bearing object:

```text
name: latest_snapshot
parameters:
  relation, key, effective_time, as_of
body:
  semantic relational skeleton
guard:
  key matches output business grain
  effective_time is valid ordering field
  cutoff is <= as_of
  tie behavior is defined
support:
  verified histories / equivalence evidence
counterexamples:
  known guard violations
```

The guard is part of the operator semantics, not metadata attached after the fact.

---

## 6. Method

### 6.1 Verified experience store

Input histories contain:

```text
natural-language request
warehouse/schema/repository context
verified SQL
execution/evaluation evidence
available contracts/metadata
```

Only verified programs provide positive abstraction support.

### 6.2 Typed relational lowering

Expose only semantics that matter for abstraction safety:

- scans/sources;
- join structure/predicates;
- filters;
- grouping and output grain;
- aggregation/distinctness;
- temporal/window semantics;
- projections/lineage;
- keys/nulls/relationship facts where known.

Avoid a large ontology initially.

### 6.3 Candidate family proposal

Use permissive mechanisms to propose recurring families:

- AST/plan similarity;
- e-graph equivalence under globally safe rules;
- anti-unification/top-down library search;
- shared type/grain;
- shared natural-language/business cues.

Candidate discovery is not the claimed novelty.

### 6.4 Guard/equivalence evidence

For each candidate family, search for a compact sufficient condition under which supporting implementations are interchangeable.

Evidence hierarchy:

1. deterministic safe relational rewrites;
2. bounded/formal SQL equivalence under supported constraints;
3. predicate/conditional reasoning over known contract facts;
4. contract-preserving differential/counterexample database instances;
5. grain/lineage/invariant consistency.

Final pairwise/family decisions are:

```text
EQUIVALENT
NOT_EQUIVALENT
UNKNOWN
```

Never force `UNKNOWN` to improve compression.

### 6.5 Joint library objective

An operator should trade off:

```text
cross-realization support
+ future composition utility
+ search/token reduction
+ guard simplicity/general coverage
- false merges
- guard violations
- parameter complexity
- uncertainty
```

Compression can be reported but is not the primary objective.

### 6.6 Inference

For a new task:

```text
question + current warehouse evidence
         ↓
candidate learned operators
         ↓
guard check
    ↙             ↘
satisfied       unknown/false
   ↓                 ↓
compose          base Text2SQL fallback
   ↓
SQL realization + normal validation
```

---

## 7. Example

Two histories implement latest valid state differently:

```text
A: ROW_NUMBER() OVER (PARTITION BY id ORDER BY effective_at DESC)
B: MAX(effective_at) GROUP BY id + join-back
```

They are not universally interchangeable. If `(id, effective_at)` can have ties, B may return multiple rows unless tie semantics are handled.

The learned operator can unify them only under a sufficient guard such as a uniqueness/tie policy. A new schema that violates this guard must not reuse the abstraction automatically.

This hard-positive/hard-negative structure is exactly what Gate A must test.

---

## 8. Claims

### C1 — Guarded Abstraction Gap

Realistic SQL workloads contain useful abstraction families recoverable under warehouse-conditioned equivalence but missed by strong syntax/fixed-theory library learners at comparable false-merge precision.

### C2 — Guard safety

Applicability guards reduce incorrect abstraction reuse on near-miss contexts.

### C3 — Composition beyond memory

Guarded operators improve held-out combinations of previously observed motifs versus strongest verified-query/structured-memory baselines under matched context.

### C4 — Joint induction, optional

Joint abstraction+guard selection improves over a composed PL pipeline that learns abstractions first and preconditions second.

C4 is explicitly optional; it must be deleted if the composed baseline matches.

---

## 9. Required baselines

### Library-learning family

- Stitch;
- babble/LLMT;
- **E-Stitch**;
- ReGAL.

### Conditional/precondition family

- fixed conditional rewrite rules;
- predicate/colored-egraph-style conditional equality;
- **Alive-Infer-style precondition inference**;
- a composed `E-Stitch/LLMT + conditional reasoning + precondition inference` baseline.

### Text2SQL reuse family

- verified-query retrieval;
- matched-context retrieval;
- AgentSM-like structured program memory;
- GATE-like execution-grounded memory;
- manual/oracle guarded semantic library.

---

## 10. Kill conditions

Kill the main SemLibSQL method thesis if any of the following holds:

1. no meaningful Guarded Abstraction Gap exists on realistic workload cases;
2. E-Stitch/babble/ReGAL with a strong global SQL theory matches abstraction recall/precision;
3. the composed conditional-PL baseline matches joint induction;
4. improvements are only alias/format/CTE normalization;
5. false semantic merges materially increase;
6. guards require motif-by-motif manual authoring;
7. verified/structured memory matches held-out composition at matched context.

Do **not** add RL, MCTS, extra agents, debate, or more memory to rescue a failed Gate A.

---

## 11. Complexity intentionally rejected

The first empirical decision does not need:

- model fine-tuning;
- RL;
- multi-agent orchestration;
- active schema POMDP;
- automatic semantic-view materialization;
- general memory architecture;
- broad ontology induction.

A small CPU/SQL-heavy controlled corpus is enough to decide whether the mechanism deserves further work.

---

## 12. Backup if the idea dies

**TemporalSQL-Drift** becomes the preferred backup: evaluate Text2SQL under changing business semantics, distinguishing valid time, knowledge time, and recomputation with current definitions. The novelty is the Text2SQL/business-semantic drift benchmark, not bitemporal memory itself.

---

## 13. Stop boundary

Further conceptual elaboration cannot answer the key question. The next step is to instantiate executable SQL/database cases and run Gate A.

**No pilot has been run.**
