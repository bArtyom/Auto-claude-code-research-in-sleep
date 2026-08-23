# SemLibSQL Seed Corpus Manifest

> Purpose: define the first 36 controlled examples before any code is written.
> These are corpus specifications, not yet generated SQL artifacts.

## 0. Three seed warehouses

### W1 — Commerce

Core relations:

- `customer_history(customer_id, status, effective_at, loaded_at)`
- `orders(order_id, customer_id, order_at, gross_amount, refund_amount, currency)`
- `customer_segment_bridge(customer_id, segment_id)`
- `fiscal_calendar(date_day, fiscal_year, fiscal_month)`

### W2 — SaaS

Core relations:

- `subscription_history(account_id, plan_id, state, valid_from, valid_to)`
- `events(event_id, account_id, event_type, event_at)`
- `account_workspace_bridge(account_id, workspace_id)`
- `calendar(date_day, fiscal_period)`

### W3 — Finance

Core relations:

- `account_state(account_id, risk_state, effective_at)`
- `ledger_entries(entry_id, account_id, posted_at, amount, entry_type, currency)`
- `account_entity_bridge(account_id, entity_id)`
- `reporting_calendar(date_day, reporting_period)`

---

# 1. Motif M1 — `latest_snapshot(key, time, as_of)`

## M1-01 / W1

- question: customer state as of month end
- realization: `ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY effective_at DESC)` + cutoff + outer filter
- gold motif: `latest_snapshot(customer_history, customer_id, effective_at, as_of)`
- positive family: M1

## M1-02 / W1

- same semantics as M1-01
- realization: `MAX(effective_at)` grouped by customer + join back
- hard positive against M1-01

## M1-03 / W2

- question: latest subscription record before reporting time
- realization: window + nested subquery
- gold motif: same abstract operator with different table/schema

## M1-04 / W2

- same semantics as M1-03
- realization: aggregate-and-join
- hard positive

## M1-05 / W3

- question: latest risk state before cutoff
- realization: precomputed `account_state_latest_before` view reference
- positive only if the view contract is supplied as authoritative metadata

## M1-06 / W3 — HARD NEGATIVE

- question: current absolute latest risk state
- realization: syntactically similar window query but **no `as_of` cutoff**
- gold motif: `absolute_latest`, not `latest_snapshot(as_of)`

---

# 2. Motif M2 — `scd2_state(key, valid_from, valid_to, as_of)`

## M2-01 / W2

- question: subscription state valid on a date
- realization: `valid_from <= as_of AND (valid_to > as_of OR valid_to IS NULL)`

## M2-02 / W2

- same semantics
- realization: max `valid_from` below cutoff + join back, under non-overlap SCD2 constraint
- equivalence depends on integrity assumption

## M2-03 / W1

- customer policy history expressed with `effective_at` + next-row boundary derived by `LEAD`
- semantic skeleton should normalize to interval state

## M2-04 / W3

- entity compliance state with explicit start/end timestamps

## M2-05 / W3

- same semantics as M2-04 via authoritative snapshot model

## M2-06 / W2 — HARD NEGATIVE

- asks whether account was **ever** active during period rather than state at one `as_of`
- similar interval predicates but different temporal semantics

---

# 3. Motif M3 — `deduplicate(relation, business_key, tie_break)`

## M3-01 / W2

- deduplicate events by `event_id`
- realization: `ROW_NUMBER() ... ORDER BY loaded_at DESC`

## M3-02 / W2

- same semantics
- realization: max load timestamp + join

## M3-03 / W1

- deduplicate orders by `order_id`
- realization: grouped max ingestion id

## M3-04 / W3

- deduplicate ledger entries by external business key
- realization: anti-join / NOT EXISTS against newer duplicate

## M3-05 / W3

- same semantics via pre-cleaned model reference

## M3-06 / W1 — HARD NEGATIVE

- deduplicate by `customer_id` instead of `order_id`
- surface shape similar but output grain changes

---

# 4. Motif M4 — `bridge_join(A, B, bridge)`

## M4-01 / W1

- customers to segments through bridge
- realization: explicit inner join chain

## M4-02 / W1

- same membership semantics
- realization: `EXISTS` semi-join because only customer identity is projected

## M4-03 / W2

- accounts to workspaces through bridge
- explicit join + distinct account

## M4-04 / W2

- same semantics via semi-join

## M4-05 / W3

- accounts to legal entities through bridge
- precomputed mapping model

## M4-06 / W3 — HARD NEGATIVE

- asks for one output row per **entity-account pair**, so duplicate collapse is incorrect
- syntactically close to membership query but different output grain

---

# 5. Motif M5 — `fiscal_period(date, calendar)`

## M5-01 / W1

- map order date to fiscal month via calendar-table join

## M5-02 / W1

- same mapping via deterministic CASE/date arithmetic for a fixed April-start calendar

## M5-03 / W2

- event date to fiscal period via calendar join

## M5-04 / W3

- posting date to reporting period via reporting calendar

## M5-05 / W3

- same semantics via precomputed `reporting_period` column with verified lineage

## M5-06 / W1 — HARD NEGATIVE

- maps to ordinary calendar month rather than fiscal month

---

# 6. Motif M6 — `recognized_revenue(period, currency_policy)`

## M6-01 / W1

- revenue = gross amount - refunds within fiscal period
- explicit expression over `orders`

## M6-02 / W1

- same semantics through verified finance model

## M6-03 / W3

- recognized ledger revenue from included entry types
- explicit filtered aggregate

## M6-04 / W3

- same semantic measure via authoritative reporting model

## M6-05 / W2

- subscription revenue represented through monthly recognized revenue table/model

## M6-06 / W1 — HARD NEGATIVE

- gross bookings without refund subtraction
- deliberately similar aggregation but different business metric

---

# 7. First composition seeds

These are not part of Gate-A clustering metrics; they define the later Gate-B split.

### History compositions

- C-H1: `latest_snapshot + active_entity`
- C-H2: `active_entity + recognized_revenue`
- C-H3: `fiscal_period + recognized_revenue`
- C-H4: `bridge_join + eligible_population`
- C-H5: `deduplicate + rolling_window`

### Held-out compositions

- C-T1: `latest_snapshot + recognized_revenue`
- C-T2: `latest_snapshot + active_entity + recognized_revenue`
- C-T3: `scd2_state + fiscal_period + recognized_revenue`
- C-T4: `deduplicate + bridge_join + eligible_population`

Every primitive in each held-out target must be independently evidenced in the history corpus, while the complete composition is absent.

---

# 8. Seed annotation rubric

For each pair of examples considered equivalent, annotate:

```yaml
pair:
  left: M1-01
  right: M1-02
label: EQUIVALENT | NOT_EQUIVALENT | UNKNOWN
scope:
  output_contract: customer_state_as_of
assumptions:
  - effective_at has a unique maximum per customer before cutoff
verification_target:
  formal: preferred
  differential_instances: required_if_formal_unknown
notes: ...
```

For hard negatives, record the **minimal semantic difference**:

- time scope;
- grain;
- identity key;
- metric definition;
- duplicate semantics;
- join multiplicity;
- population definition.

---

# 9. What these 36 seeds are intended to prove

The seed set deliberately creates two axes:

1. same semantics / very different syntax;
2. similar syntax / importantly different semantics.

If SemLibSQL cannot beat generic syntactic/refactoring baselines on this controlled set, scaling to hundreds of programs is not justified.

## 10. Next artifact after review

Convert these specifications into a machine-readable manifest and actual verified SQL instances only after the corpus design is accepted. Until then, avoid coding around assumptions that may change.