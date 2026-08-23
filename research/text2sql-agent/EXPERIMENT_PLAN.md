# EviSQL-Agent Experiment Plan

## 1. Goal

Test whether **evidence-gated, counterexample-guided verification** can reduce silent semantic errors in Text2SQL compared with single-shot and ordinary execute-and-repair agents, under matched inference budgets.

The first milestone is intentionally narrow: demonstrate that targeted probes can distinguish between multiple executable-but-disagreeing SQL candidates often enough to justify a full agent architecture.

## 2. Core hypotheses

### H1 — Counterexample probes beat generic self-critique

Given two or more SQL candidates that all execute but return different answers, a verifier that actively designs a discriminative database probe will select the semantically correct candidate more often than:

- majority vote,
- LLM self-critique,
- “pick the SQL that looks most plausible”,
- execution-success filtering alone.

### H2 — Evidence gates reduce silent errors

At matched token/tool budget, an agent that may finalize only after explicit evidence gates pass will have a lower silent semantic error rate than a ReAct-style agent that stops when execution appears successful.

### H3 — Adaptive information acquisition improves efficiency

A policy that inspects only evidence needed to resolve current uncertainty will use fewer schema/value/tool observations than full-schema prompting at similar task success.

## 3. Experimental stages

## Stage A — Disagreement benchmark construction

Create a small diagnostic set from 50–100 questions.

For each question:

1. Generate 3–5 SQL candidates with controlled diversity.
2. Execute all valid candidates.
3. Retain cases where at least two candidates execute but disagree in output.
4. Label the semantic difference between candidates.
5. Record the smallest evidence/probe that would distinguish them.

Target initial dataset: **30–60 disagreement cases**.

### Failure labels

Use a fixed taxonomy:

- wrong join path
- duplicate amplification
- missing DISTINCT
- wrong aggregation grain
- wrong date/time field
- wrong filter value/entity mapping
- incorrect null treatment
- incorrect grouping
- wrong denominator
- wrong ordering/top-k semantics
- dialect-specific semantic behavior
- hidden business-rule mismatch
- other

This diagnostic set becomes a reusable verifier benchmark.

## Stage B — Probe-generation pilot

Input to verifier:

- natural-language question
- selected schema metadata
- candidate SQL A/B (or A/B/C)
- database tool access
- no gold SQL and no gold result

Verifier may issue at most K diagnostic probes, e.g. K ∈ {1, 3, 5}.

### Probe types

- cardinality check
- duplicate check
- unmatched join row check
- value distribution
- null distribution
- date range inspection
- candidate-result diff localization
- partition consistency check
- subset/superset metamorphic check
- uniqueness/functional-dependency check

### Baselines

- random candidate
- majority result/candidate voting
- LLM judge without DB probes
- execute-and-self-reflect
- generic unit-test generation
- discriminative probe generation (ours)

### Primary metric

`selection_accuracy = correctly selected semantic candidate / disagreement cases`

Secondary:

- probe success rate
- probes per resolved case
- DB calls per case
- token cost
- percent of errors localized to correct failure class

## Stage C — End-to-end EviSQL-Agent

### Components

1. **IntentAnalyzer**
   - extracts requested metric, population, filters, time window, grouping, ranking
   - flags decision-relevant ambiguity

2. **EvidencePlanner**
   - defines required evidence IDs before finalization

3. **SchemaExplorer**
   - retrieves tables/columns/docs selectively

4. **ValueExplorer**
   - samples/searches values only when entity mapping is uncertain

5. **CandidateGenerator**
   - generates N diverse candidates

6. **StaticVerifier**
   - parser, object existence, forbidden operations, dialect check

7. **ExecutionSandbox**
   - read-only execution, timeout, row/scan limits

8. **CounterexampleAgent**
   - designs probes that separate plausible candidates

9. **SemanticReviewer**
   - independent/fresh reviewer checks candidate against question + evidence ledger

10. **EvidenceGate**
   - PASS / WARN / FAIL / BLOCKED
   - on FAIL loops with specific missing evidence
   - on BLOCKED asks clarification or abstains

## 4. Evidence ledger schema

Suggested JSONL/JSON representation:

```json
{
  "question_id": "...",
  "evidence": [
    {
      "id": "schema.customer.orders_join",
      "type": "join",
      "claim": "orders.customer_id joins customers.id",
      "source": "foreign_key_metadata",
      "observation": "orders.customer_id -> customers.id",
      "confidence": 1.0
    },
    {
      "id": "probe.no_duplicate_amplification",
      "type": "counterexample_probe",
      "claim": "join does not duplicate order grain",
      "query": "...",
      "observation": "ratio=1.00",
      "confidence": 0.99
    }
  ],
  "candidate_status": [
    {"candidate_id": "sql_1", "status": "rejected", "reason_evidence_ids": ["probe.duplicate_amplification"]},
    {"candidate_id": "sql_2", "status": "survives", "reason_evidence_ids": ["schema.customer.orders_join", "probe.no_duplicate_amplification"]}
  ],
  "gate": {
    "status": "PASS",
    "required": ["schema", "entity", "semantic", "execution"],
    "missing": []
  }
}
```

The key rule: free-form rationale does not count as evidence unless anchored to an observation/tool result.

## 5. Termination policy

A candidate can finalize only if:

- referenced tables/columns exist;
- unresolved intent ambiguity is below threshold;
- required entity/value mappings are supported;
- SQL executes or compiles in the target dialect;
- no known candidate disagreement remains unresolved;
- no high-severity counterexample remains;
- cost budget is respected;
- independent semantic review is non-negative.

If the budget is exhausted with missing evidence, return `BLOCKED` rather than a confident SQL answer.

## 6. Datasets and benchmark order

### Phase 1 — cheap local development

- SQLite/DuckDB subset of Spider/BIRD-like data
- Mini-Interact if available in a convenient local format

Purpose: iterate on probes and ledger quickly.

### Phase 2 — standard Text2SQL

- BIRD dev/mini variants
- LiveSQLBench lite/full development setting where permitted

Purpose: compare semantic accuracy and cost.

### Phase 3 — enterprise workflow

- Spider 2.0 Lite/Snow
- Spider2-DBT for repository-aware extension

Purpose: establish scaling to large schemas and multi-step workflows.

## 7. Matched-budget protocol

Agent systems are easy to make look better simply by spending more calls. Therefore report at least three fixed budgets:

- **Small:** e.g. 1 generator call + ≤2 DB probes
- **Medium:** ≤5 model calls + ≤5 DB probes
- **Large:** ≤10 model calls + ≤10 DB probes

Also report unconstrained best-effort as a separate curve, never mixed with matched-budget comparisons.

Plot/compare:

- success vs tokens
- success vs model calls
- success vs DB probes
- silent-error rate vs cost
- risk-coverage curve for abstention

## 8. Ablation matrix

| Variant | Ledger | Diverse candidates | DB probes | Counterexample targeting | Independent reviewer | Clarification |
|---|---:|---:|---:|---:|---:|---:|
| Single-shot | No | No | No | No | No | No |
| Execute-repair | No | No | Yes | No | No | No |
| Multi-candidate | No | Yes | Yes | No | No | No |
| + generic tests | No | Yes | Yes | No | No | No |
| + evidence ledger | Yes | Yes | Yes | No | No | No |
| + counterexamples | Yes | Yes | Yes | Yes | No | No |
| + independent review | Yes | Yes | Yes | Yes | Yes | No |
| Full EviSQL | Yes | Yes | Yes | Yes | Yes | Yes |

## 9. Statistical plan

For the first pilot, use paired evaluation because all systems see the same questions.

Report:

- bootstrap confidence intervals for success-rate differences
- paired McNemar test for binary task success where appropriate
- per-failure-class breakdown
- mean/median cost with heavy-tail percentiles

Avoid overclaiming from a 30-case diagnostic pilot; treat it as mechanism validation, not benchmark SOTA evidence.

## 10. Implementation skeleton

Suggested project layout:

```text
research/text2sql-agent/
  IDEA_REPORT.md
  EXPERIMENT_PLAN.md
  ARIS_INTEGRATION.md
  prototype/
    agents/
      intent.py
      explorer.py
      generator.py
      counterexample.py
      reviewer.py
      gate.py
    tools/
      db.py
      schema.py
      sql_static.py
    eval/
      runner.py
      metrics.py
      failure_taxonomy.py
    schemas/
      evidence_ledger.schema.json
    configs/
      pilot.yaml
```

Do not implement the full stack before Stage B falsifies or supports the core counterexample hypothesis.

## 11. First 48-hour pilot checklist

- [ ] Pick 30–50 benchmark examples with local DB access.
- [ ] Generate 3 diverse SQL candidates per example.
- [ ] Identify executable disagreement cases.
- [ ] Manually label failure category and correct candidate.
- [ ] Build a read-only DB probe tool with row/time limits.
- [ ] Prompt one verifier to generate one discriminative probe per case.
- [ ] Compare against no-probe LLM judge.
- [ ] Calculate selection accuracy and probe cost.
- [ ] Inspect failure cases qualitatively.
- [ ] Decide GO / PIVOT / STOP for the counterexample mechanism.

## 12. Go/no-go criteria

Proceed to a full agent paper if, on the disagreement set:

- discriminative probes produce a clear (>10 absolute points is a useful heuristic target, not a formal threshold) improvement over no-probe judging;
- gains are not restricted to one failure type;
- most probes are cheap enough to run under realistic warehouse constraints;
- verifier improvements survive a fresh model/reviewer ablation.

Pivot toward selective clarification if ambiguity dominates and DB probes cannot distinguish candidates. Pivot toward failure-memory/repository context if errors are mostly caused by undocumented organization-specific semantics.
