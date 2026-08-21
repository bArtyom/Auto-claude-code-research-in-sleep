# ARIS Integration Design: Text2SQL / SQL-Agent Research

## 1. Purpose

This note maps the Text2SQL-agent research direction onto the existing ARIS architecture without prematurely adding a large new subsystem.

The recommendation is to treat SQL-agent research as a **domain research package** first, and only promote it into reusable ARIS skills after the pilot demonstrates a durable workflow.

## 2. Reusable ARIS concepts

ARIS already provides several primitives that are directly useful:

- executor/reviewer separation
- cross-model independent review
- evidence-bearing gates
- resumable run state
- fan-out for candidate diversity
- artifact contracts
- experiment integrity audits
- final PASS/WARN/FAIL/BLOCKED semantics

The SQL-agent project should reuse these concepts rather than invent parallel orchestration conventions.

## 3. Proposed domain artifacts

```text
research/text2sql-agent/
  IDEA_REPORT.md
  EXPERIMENT_PLAN.md
  ARIS_INTEGRATION.md
  runs/
    <run_id>/
      QUESTION.json
      CANDIDATES.json
      EVIDENCE_LEDGER.json
      PROBE_LOG.jsonl
      REVIEW.json
      FINAL.json
```

Suggested run artifact responsibilities:

| Artifact | Role |
|---|---|
| `QUESTION.json` | normalized task, dialect, DB locator, user constraints |
| `CANDIDATES.json` | candidate SQL programs and generation metadata |
| `EVIDENCE_LEDGER.json` | machine-readable evidence supporting/rejecting candidates |
| `PROBE_LOG.jsonl` | every schema/value/execution/counterexample action |
| `REVIEW.json` | fresh reviewer verdict with model identity and evidence references |
| `FINAL.json` | selected SQL, answer, confidence, gate status, cost |

## 4. Proposed skill sequence after MVP validation

Only if the pilot is successful, consider adding these skills:

### `/sql-agent-pilot`

Input: benchmark/config or one database task.

Responsibilities:

1. generate diverse candidates;
2. execute under read-only sandbox constraints;
3. find executable disagreement;
4. generate discriminative probes;
5. write an evidence ledger;
6. invoke a fresh reviewer;
7. emit PASS/BLOCKED and metrics.

### `/sql-failure-mine`

Input: run directory or batch of benchmark runs.

Responsibilities:

- classify first failure point;
- cluster structural error patterns;
- identify missing evidence that would have prevented each failure;
- generate a failure-memory dataset.

### `/sql-agent-eval`

Input: config + benchmark split.

Responsibilities:

- enforce matched inference budgets;
- run baseline and ablation variants;
- aggregate task success, silent error, tokens, DB calls and latency;
- produce reproducible result tables.

### `/sql-agent-review`

Fresh independent reviewer that receives:

- question/task
- candidate SQL
- database/schema access as allowed
- evidence ledger

It should **not** receive the generator's chain-of-thought or persuasive summary.

## 5. Reviewer independence contract

For SQL, reviewer independence matters because the most dangerous errors are plausible, executable semantic errors.

Recommended rules:

1. Generator and semantic reviewer should be different model families when possible.
2. Reviewer sees artifacts and observations, not generator rationale.
3. Reviewer must cite evidence IDs for each rejection/acceptance reason.
4. Same-family review is recorded as provisional.
5. If the reviewer cannot validate a critical business semantic, return `BLOCKED`, not guessed acceptance.

This closely follows ARIS's existing review philosophy.

## 6. SQL-specific experiment integrity rules

Add domain constraints beyond ordinary ML experiment integrity:

- benchmark gold SQL/result must never be exposed to the inference agent;
- DB tool should be read-only;
- block DDL/DML by default (`INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER`, etc.);
- enforce query timeout and row/byte limits;
- record every executed probe;
- separate oracle-table / oracle-schema experiments from non-oracle settings;
- record engine/dialect/version;
- record whether sample values, comments, docs or lineage were available;
- never treat “query executed” as semantic PASS;
- do not cherry-pick candidate generations across repeated runs without reporting selection protocol.

## 7. Suggested gate schema

Use ARIS-style verdict states:

```json
{
  "status": "PASS | WARN | FAIL | BLOCKED | ERROR | NOT_APPLICABLE",
  "checks": {
    "intent": "PASS",
    "schema": "PASS",
    "entity_mapping": "PASS",
    "execution": "PASS",
    "semantic_counterexamples": "PASS",
    "cost": "PASS",
    "independent_review": "PASS"
  },
  "missing_evidence_ids": [],
  "rejected_candidate_ids": [],
  "review_independence": "cross-family"
}
```

A missing critical check should cause `BLOCKED`, not a default PASS.

## 8. Candidate fan-out pattern

Candidate diversity should be controlled, not just temperature noise.

Useful candidate roles:

- minimal direct SQL
- decomposition/CTE-heavy SQL
- conservative join SQL
- aggregation-first SQL
- alternative date/entity interpretation

Each candidate should be generated independently enough that correlated mistakes are reduced. The system should log generation role and model.

## 9. Evidence gate pseudocode

```text
normalize(question)
required = plan_required_evidence(question)

while budget_remaining:
    gather cheapest missing evidence
    candidates = generate_or_repair_candidates(state)
    discard static/execution failures

    if multiple plausible candidates disagree:
        probe = design_discriminative_probe(candidates, state)
        observation = execute_probe(probe)
        update_ledger(observation)
        continue

    verdict = independent_review(candidate, ledger)
    if all critical gates PASS:
        finalize

    if missing evidence is user-only:
        ask targeted clarification

return BLOCKED
```

The key design choice is that the loop is driven by **missing evidence**, not generic “reflection”.

## 10. Research integration with existing ARIS workflow

A full research run could eventually look like:

```text
/idea-discovery "agentic Text2SQL with evidence-gated verification"
  ↓
/experiment-bridge
  ↓
/sql-agent-pilot
  ↓
/sql-failure-mine
  ↓
/analyze-results
  ↓
/experiment-audit
  ↓
/auto-review-loop
  ↓
/paper-writing
```

No new top-level research pipeline is required initially. The domain skills can compose into existing W1/W1.5/W2/W3 structure.

## 11. What not to build yet

Avoid these until the mechanism pilot is positive:

- a large custom UI;
- a persistent vector-memory service;
- RL training infrastructure;
- multi-database production connectors;
- a new full benchmark framework;
- a dozen specialist agents;
- a large synthetic-data pipeline.

The scientific bottleneck is first to answer: **can targeted evidence acquisition and counterexample probes reliably detect semantic SQL errors that ordinary agents miss?**

## 12. Promotion criteria into `skills/`

Promote the domain workflow from `research/text2sql-agent/` into ARIS skills only after:

1. at least one reproducible benchmark runner exists;
2. evidence ledger schema stabilizes;
3. at least one baseline and one ablation can be run end-to-end;
4. tool safety constraints are tested;
5. reviewer receipt semantics are defined;
6. the core mechanism shows meaningful pilot gains.

Until then, keeping this work in `research/` avoids polluting the stable 82-skill catalog with an unvalidated workflow.
