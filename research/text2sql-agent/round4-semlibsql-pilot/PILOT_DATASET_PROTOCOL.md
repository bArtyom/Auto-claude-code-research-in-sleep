# SemLibSQL Pilot Dataset Protocol

> Workflow role: experiment design / pre-registration
> Goal: construct the smallest corpus that can falsify the SemLibSQL mechanism.

## 1. What the pilot must isolate

The pilot is not a miniature full Text2SQL benchmark. It isolates one mechanism:

> semantic canonicalization should reveal reusable motifs hidden by syntactic variation.

If the corpus does not contain strong surface variation for the same semantics, the pilot is invalid.

---

## 2. Unit of annotation

Each verified program record should contain:

```yaml
id: unique-record-id
database_id: warehouse/project
sql_dialect: duckdb|postgres|snowflake-like|bigquery-like
question: natural-language request
sql: verified SQL
motif_ids: [one-or-more gold semantic motifs]
composition_id: canonical set/order of motifs
output_grain: optional gold label
entity_types: optional gold labels
verification: execution|formal|manual
generation_family: window|aggregate-join|cte|nested|precomputed-model|other
near_duplicate_group: leakage-control id or null
```

Gold motif labels exist **only for evaluation**, not necessarily for induction.

---

## 3. Initial motif inventory

Prefer motifs that can be realized by materially different SQL structures.

### High-priority motifs

1. `latest_snapshot(key, time, as_of)`
   - ROW_NUMBER / QUALIFY
   - MAX(time) aggregate + join
   - DISTINCT ON
   - precomputed latest-state model

2. `scd2_state(key, valid_from, valid_to, as_of)`
   - interval predicate
   - max valid_from under cutoff
   - current flag + temporal fallback

3. `dedup_events(entity_key, event_identity)`
   - DISTINCT
   - ROW_NUMBER partition
   - aggregate/min id
   - anti-join duplicate removal

4. `bridge_join(A, B, bridge)`
   - explicit bridge relation
   - nested semi-join
   - pre-aggregated mapping model

5. `eligible_population(policy)`
   - positive inclusion predicate
   - exclusion anti-join
   - status lookup table

6. `recognized_revenue(period, currency_policy)`
   - transaction aggregation
   - precomputed finance model
   - refund subtraction / sign normalization

7. `fiscal_period(date, calendar)`
   - date arithmetic
   - fiscal calendar table join
   - CASE-based mapping

8. `rolling_window(metric, width)`
   - window frame
   - self-join time range
   - correlated subquery

### Optional motifs if corpus size permits

- retention cohort
- funnel stage order
- top-k with ties
- latest non-null attribute

Avoid motifs whose variants differ only in aliases or whitespace; those do not test the thesis.

---

## 4. Corpus size

### Minimum viable pilot

- 8 motifs
- 3 projects/schemas
- 12–20 instances per motif
- 200–300 verified SQL programs total

### Better pilot

- 10–12 motifs
- 4 projects
- 20–30 instances per motif
- 350–500 programs

No model training is required for Gate A.

---

## 5. Data construction sources

Use a mixture of:

1. hand-constructed synthetic schemas with explicit semantic contracts;
2. public benchmark schemas where motifs can be verified;
3. generated SQL variants that are formally/execution verified equivalent;
4. repository/dbt-style variants where the same motif appears through a precomputed model.

The purpose is controlled semantic diversity, not benchmark realism at any cost.

---

## 6. Generating hard positive pairs

For each motif, deliberately create pairs that generic syntax miners should find difficult:

### Example: latest snapshot

A — window formulation:

```sql
SELECT *
FROM (
  SELECT t.*, ROW_NUMBER() OVER (
    PARTITION BY customer_id ORDER BY effective_at DESC
  ) AS rn
  FROM customer_history t
  WHERE effective_at <= :as_of
) x
WHERE rn = 1;
```

B — aggregate + join:

```sql
SELECT h.*
FROM customer_history h
JOIN (
  SELECT customer_id, MAX(effective_at) AS mx
  FROM customer_history
  WHERE effective_at <= :as_of
  GROUP BY customer_id
) m
ON h.customer_id = m.customer_id
AND h.effective_at = m.mx;
```

C — DISTINCT ON realization in Postgres-like SQL.

D — direct scan of a verified precomputed `customer_latest_snapshot` model.

These should share a gold semantic motif while being far apart syntactically.

---

## 7. Hard negative pairs

The pilot also needs syntactically similar SQL with different semantics, otherwise aggressive canonicalization can appear better than it is.

Examples:

- latest snapshot before `as_of` vs absolute latest snapshot;
- deduplicate by event id vs deduplicate by customer;
- active customer at period end vs ever active during period;
- rolling 30-day sum vs calendar-month sum;
- bridge join with one-to-many preservation vs bridge join followed by distinct entity collapse.

These are the primary false-merge tests.

---

## 8. Verification hierarchy

Assign each positive-equivalence relation an evidence level:

### V3 — formal proof

Use SQLSolver/Cosette-supported fragments when possible.

### V2 — differential execution on adversarial instances

Generate small database instances designed to separate common semantic confusions.

### V1 — expert/manual contract

Use only when formal/differential verification is impractical.

The pilot must report metrics separately by evidence level.

---

## 9. Adversarial database instances

For each motif, generate edge-case instances.

### latest_snapshot

- tied timestamps
- future rows
- duplicate rows
- null times
- single-key and multi-key cases

### dedup

- exact duplicates
- duplicate business key with different event ids
- null keys

### bridge joins

- one-to-many bridge
- many-to-many bridge
- missing mappings
- duplicate bridge rows

### fiscal period

- year boundary
- non-January fiscal start
- leap day

Behavior signatures should include these tests rather than only the original warehouse instance.

---

## 10. Held-out composition split

After Gate A, build composition tasks with the rule:

- every primitive in the target composition appears individually in history;
- at least some pairwise combinations appear;
- the complete target composition never appears;
- no near-duplicate full SQL exists;
- NL wording is independently paraphrased.

Example:

```text
history:
latest_snapshot
active_entity
recognized_revenue
latest_snapshot + active_entity
active_entity + recognized_revenue

held out:
latest_snapshot + recognized_revenue
latest_snapshot + active_entity + recognized_revenue
```

This tests abstraction reuse rather than template memorization.

---

## 11. Baseline-ready representations

Persist multiple views of each program so all baselines receive identical source data:

- raw SQL tokens;
- normalized aliases/literals;
- parsed AST;
- logical relational plan;
- schema-grounded plan;
- semantic-plan representation;
- execution/behavior signature.

No baseline should be handicapped by receiving less source information unless that difference defines the ablation.

---

## 12. Pilot success criteria

A useful Gate-A result requires all of:

1. semantic method improves hard-positive recall over syntactic library learning;
2. false-merge rate on hard negatives remains acceptably low;
3. improvement is not limited to trivial alias/format rewrites;
4. discovered abstractions have coherent parameterization;
5. at least several high-value abstractions cross SQL realization families.

If only clustering purity improves on easy examples, the mechanism is not enough for a paper.