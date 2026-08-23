# SemLibSQL Canonicalization Design

> Workflow role: method refinement
> Principle: **false merges are more dangerous than missed merges**.

## 1. Design objective

Map heterogeneous SQL implementations into conservative semantic equivalence classes suitable for library induction.

The system is not a complete SQL equivalence prover. It should return one of:

```text
EQUIVALENT
NOT_EQUIVALENT
UNKNOWN
```

`UNKNOWN` is a valid and expected outcome.

---

## 2. Layered canonicalization pipeline

### Layer 0 — syntax hygiene

Normalize only semantics-preserving surface variation:

- formatting;
- aliases;
- literal placeholders where allowed;
- CTE naming;
- commutative predicate ordering;
- projection ordering when output contract permits.

This is the baseline, not the novelty.

### Layer 1 — relational logical plan

Convert SQL into a relational plan exposing:

- scan;
- filter;
- project;
- join/semi-join/anti-join;
- group/aggregate;
- window;
- distinct;
- order/limit;
- subquery dependencies.

Normalize safe relational identities conservatively.

### Layer 2 — schema-grounded semantics

Attach:

- relation/entity identity;
- PK/FK or inferred relationship;
- uniqueness constraints;
- output grain;
- column semantic role where known;
- time role;
- lineage to outputs.

This layer allows the system to distinguish, for example, `COUNT(order_id)` from `COUNT(DISTINCT customer_id)` even when both are legal aggregates.

### Layer 3 — motif-level semantic skeleton

Rewrite recognized relational structures into parameterized semantic skeletons when evidence is sufficient.

Examples:

```text
argmax_per_key(relation, key, time, cutoff)
```

```text
interval_state(relation, key, valid_from, valid_to, as_of)
```

```text
deduplicate(relation, business_key, tie_break)
```

This representation is what the library learner consumes.

### Layer 4 — equivalence evidence

Two candidate skeletons may be merged only when supported by one or more of:

1. formal equivalence proof on supported fragments;
2. safe rewrite derivation;
3. differential execution over discriminating generated instances;
4. matching lineage/grain/invariant signatures;
5. expert contract for pilot-only labels.

---

## 3. Formal-equivalence integration

Use SQLSolver/Cosette as **oracles where applicable**, not as full-system dependencies.

Possible policy:

```text
if formal prover supports pair:
    EQ -> strong merge evidence
    NEQ -> forbid merge
    UNKNOWN/TIMEOUT -> continue to other evidence
else:
    continue
```

Formal proof should be logged separately from empirical behavioral agreement.

---

## 4. Behavioral signatures

A behavior signature is not just the result on the original database. That risks accidental equivalence.

Instead evaluate candidate programs on a small suite of generated discriminating instances.

Signature fields may include:

- output cardinality;
- key multiplicities;
- selected row identities;
- aggregate values;
- null behavior;
- monotonicity under controlled changes;
- response to duplicates;
- response to boundary dates.

A pair matching only on the original instance should remain `UNKNOWN` unless another evidence source supports equivalence.

---

## 5. Grain as a first-class invariant

Many silent SQL errors are grain mismatches. Every plan should infer or annotate an approximate output grain:

```text
Grain<Customer>
Grain<Order>
Grain<Customer, Month>
Grain<Account, SnapshotDate>
```

Merging abstractions with incompatible grain is forbidden.

This simple constraint may produce a strong precision gain with little complexity.

---

## 6. Time semantics as a first-class invariant

Temporal motifs are high-value because SQL surface implementations vary widely.

Represent distinctions such as:

```text
absolute_latest
latest_before(as_of)
state_valid_at(as_of)
latest_non_null_before(as_of)
window_over_event_time(width)
window_over_calendar_period(period)
```

Do not collapse these even if they agree on current data.

---

## 7. Join semantics

Track at minimum:

- inner / left / semi / anti;
- expected cardinality relation where known;
- whether duplicates are preserved or collapsed;
- whether bridge traversal changes entity grain.

A many-to-many join followed by `DISTINCT` may be semantically equivalent to a semi-join for one projection but not for another. Context matters; equivalence must therefore be attached to the relevant output contract, not blindly global.

---

## 8. Contextual equivalence

Some SQL fragments are equivalent only under assumptions:

- uniqueness constraints;
- non-null keys;
- referential integrity;
- exhaustive status domains;
- authoritative precomputed view contracts.

Store equivalence as:

```yaml
lhs: ...
rhs: ...
status: EQUIVALENT
assumptions:
  - customer_id unique in dimension_customer
  - effective_at non-null
scope:
  database: demo_finance
  output_contract: customer_level_snapshot
```

This prevents learned primitives from becoming falsely universal.

---

## 9. Candidate abstraction mining

After canonicalization, use multiple miners:

- anti-unification;
- Stitch-like top-down compression;
- e-graph-aware anti-unification;
- simple frequent-subgraph mining.

The research question is not which miner is best. Keep the miner simple initially and isolate whether the **semantic representation** changes what can be discovered.

---

## 10. Primitive scoring

For a candidate abstraction `a`, score approximately:

```text
Score(a) =
  + support_across_experiences
  + cross-realization_diversity
  + compression_gain
  + held-out_predictive_utility
  + semantic_purity
  - parameter_count_penalty
  - assumption_complexity
  - false_merge_risk
  - context_specificity_penalty
```

For Gate A, do not tune a complicated learned score. Use transparent weighted components or report a Pareto frontier.

---

## 11. Safety rule for learned primitives

A primitive should have:

- typed parameters;
- supporting examples;
- known counterexamples;
- assumptions;
- scope;
- verification evidence;
- fallback behavior.

The generator may use the primitive only when its current context satisfies the stored assumptions with adequate confidence.

---

## 12. Key ablations

### No semantic grounding

Raw/normalized AST only.

### No formal equivalence

Safe rewrites + behavior signatures only.

### No behavior signatures

Logical/typed normalization only.

### No grain information

Tests whether grain is carrying most of the benefit.

### No time-role information

Tests whether temporal motifs are the main source of gains.

### No schema assumptions

Tests portability versus warehouse specificity.

---

## 13. Kill condition

If a ReGAL/Stitch-like learner over normalized ASTs discovers abstractions with comparable hard-positive recall, purity, and downstream utility, this canonicalization stack is unjustified.

The right scientific response is to kill or dramatically simplify the method, not add more layers.