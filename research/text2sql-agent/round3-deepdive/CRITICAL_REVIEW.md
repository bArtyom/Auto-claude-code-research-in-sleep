# Round 3 Critical Review — SemLibSQL

> **Date:** 2026-08-23  
> **Workflow role:** `research-review` equivalent fallback  
> **Status:** **PROVISIONAL — INTERNAL RED TEAM ONLY**  
> The ARIS `research-review` skill requires a fresh independent reviewer backend. That backend is not exposed in this connector session. This document must **not** be treated as cross-model acceptance.

```yaml
review_independence: same-family/internal
external_reviewer: unavailable
acceptance_status: provisional
formal_verdict: REVIEW_UNAVAILABLE
```

## 1. Mock senior-review summary

SemLibSQL proposes to learn a warehouse-specific typed semantic operator library from previously verified Text2SQL solutions. The method canonicalizes syntactically diverse SQL into semantic plans, mines recurring parameterized abstractions, validates those abstractions, and exposes them as primitives for later query generation. The central evaluation claim is that induced operators should outperform episodic query retrieval/memory on held-out **compositions** of previously observed semantic motifs.

The proposal is substantially cleaner than the earlier umbrella system. It targets a real gap between (a) Text2SQL memory systems that retain solved traces and (b) program-library-learning systems that invent reusable abstractions. However, in its current conceptual form the contribution remains vulnerable to the criticism that it is **DreamCoder/Stitch/LILO applied to SQL**, with recent agent-skill work making the general narrative even less novel.

The paper is only strong if it demonstrates a domain-specific mechanism that ordinary program compression cannot recover and if the held-out compositional evaluation rules out nearest-neighbor replay.

---

## 2. Strengths

### S1 — Clean distinction from query memory

The question "Can an SQL agent learn a better *language* rather than a larger *memory*?" is crisp and scientifically meaningful.

### S2 — Falsifiable compositional claim

A motif-composition split can directly test whether the system learned reusable primitives rather than examples.

### S3 — Strong connection to real enterprise structure

Enterprise analytics repeatedly reuses hidden business transformations such as SCD snapshots, fiscal periods, deduplication, entity eligibility, bridge joins, and governed metrics.

### S4 — Can be piloted without expensive model training

The first experiment can use fixed LLMs and offline library induction. This is an excellent property for idea validation.

### S5 — Negative results would still be informative

If verified-query retrieval matches learned abstractions, that is evidence that abstraction invention adds little beyond episodic memory in modern Text2SQL.

---

## 3. Major concerns

### C1 — Algorithmic novelty may collapse to existing library learning

**Severity: critical.**

If the implementation is:

```text
parse SQL AST -> run Stitch -> name functions with an LLM
```

then the paper is too incremental.

#### What would satisfy the concern

Show that SQL requires a distinct **semantics-aware equivalence layer**. For example, the same motif appears as window functions, correlated subqueries, `DISTINCT ON`, engine-specific `QUALIFY`, or precomputed dbt models. The proposed canonicalizer should map these to a common typed semantic representation before abstraction mining.

Required ablation:

```text
syntactic AST mining
vs
safe-rewrite canonicalization
vs
behavior/evidence-aware semantic canonicalization
```

If semantic canonicalization does not materially change abstraction quality or downstream composition accuracy, the algorithmic novelty is weak.

---

### C2 — "Semantic operator" may just be author interpretation

**Severity: critical.**

A compression algorithm can invent functions that save tokens but have no coherent business meaning.

#### Required evidence

For each learned primitive, measure:

- semantic purity across its instances;
- parameter consistency;
- schema/entity/grain consistency;
- behavior under controlled substitutions/interventions;
- human interpretability on an audited sample;
- whether independently generated instances satisfy the same invariants.

A library that compresses 30% but mixes distinct business meanings is not a semantic library.

---

### C3 — Composition benchmark could be synthetic and easy to game

**Severity: critical.**

A hand-built A/B/C split can accidentally encode the answer in operator names or schemas.

#### Required design

Use two tiers:

1. **controlled motif benchmark** for causal isolation;
2. **enterprise-like/open real workload** for external validity.

For the controlled set, hide full-program near duplicates and randomize physical implementations. For the real set, manually audit a subset of repeated semantic motifs and split chronologically or by composition.

---

### C4 — Strong memory baselines may erase the gain

**Severity: high.**

Recent Text2SQL memory/crystallization work is strong. A long context containing verified examples may already solve most future tasks.

#### Required baselines

- exact/nearest SQL retrieval;
- embedding retrieval over NL + SQL;
- structured semantic trace retrieval (AgentSM-like);
- verified corrected-query memory (Crystallization-like);
- retrieval with the same total token budget as SemLibSQL;
- manual semantic-layer/oracle primitive library.

If SemLibSQL only beats a stateless baseline, the paper fails.

---

### C5 — Library growth may create negative transfer

**Severity: high.**

An abstraction learned from ten examples may be subtly wrong on the eleventh because business rules are context-dependent.

Examples:

- `active_customer` differs by business unit;
- fiscal calendar differs by region;
- deduplication key changes over time;
- revenue definition depends on reporting purpose.

#### Required safeguards and metrics

- typed scope/context constraints on primitives;
- confidence/evidence receipts;
- explicit fallback to raw reasoning;
- negative-transfer rate;
- per-primitive error/blast-radius analysis.

Do not force library use.

---

### C6 — Evaluation based only on execution accuracy is insufficient

**Severity: high.**

Two SQL programs can execute and return the same result on one database instance while differing semantically.

Use:

- multiple database perturbations / synthetic discriminating instances;
- execution result plus semantic checks;
- invariants / output grain;
- structural/behavioral equivalence where possible;
- human audit for ambiguous cases.

---

### C7 — Cross-dialect transfer could distract from the core paper

**Severity: medium.**

Cross-dialect lowering is attractive but may turn the project into a compiler paper. Keep it as one diagnostic experiment, not the main thesis.

---

### C8 — The method could quietly become an LLM-heavy multi-agent system

**Severity: high.**

Avoid adding planner, critic, verifier, ontology agent, memory agent, and view-creation agent around the core. That would recreate the complexity that the Round-2 review tried to remove.

The minimal adequate system should be:

```text
verified experiences
-> semantic normalization
-> abstraction induction
-> abstraction validation
-> library-aware generation
```

Everything else is optional.

---

## 4. Required experiments ranked by acceptance lift

### E1 — The anchor experiment: held-out composition

**Priority: must-run.**

Train/library-build on motifs and partial compositions; test on unseen compositions with retrieval leakage controlled.

This is the experiment most directly tied to the thesis.

### E2 — Syntactic vs semantic abstraction induction

**Priority: must-run.**

If ordinary Stitch-style AST compression performs equally well, the SQL-specific method is unnecessary.

### E3 — Memory baseline stress test

**Priority: must-run.**

Compare with the strongest verified-query memory under equal context/token budget.

### E4 — Abstraction quality audit

**Priority: must-run.**

Measure purity, interpretability, parameter consistency, and intervention behavior.

### E5 — Negative-transfer / forced-use deletion test

**Priority: must-run.**

Compare optional library routing vs always-use library. Demonstrate the system can avoid stale/inapplicable abstractions.

### E6 — One real/enterprise-like workload

**Priority: must-run before strong paper claims.**

Controlled synthetic experiments alone are insufficient.

### E7 — Cross-dialect reuse

**Priority: optional.**

Useful if the semantic IR naturally supports it, but delete if it slows the core story.

---

## 5. Claims matrix

| Experimental outcome | Allowed claim |
|---|---|
| SemLibSQL > memory on held-out composition; semantic canonicalization > syntactic mining | Strong: semantic library induction improves compositional Text2SQL beyond memory |
| SemLibSQL > memory, but semantic canonicalization ≈ syntactic mining | Weaker: library induction helps, but SQL-specific mechanism not justified |
| SemLibSQL ≈ memory, but lower inference cost | Efficiency/system claim only; novelty reduced |
| SemLibSQL < memory | Core thesis falsified; stop or study why abstraction hurts |
| Synthetic gains, no real-workload gains | Benchmark/method limited; avoid broad enterprise claims |
| High gains but high negative transfer | Unsafe; need scoped routing/versioning before publication |
| Manual library strongly beats induced library | Shows semantic abstraction is useful but automatic induction remains unsolved |

---

## 6. Mock score

**Novelty:** 6.5–7/10 conditional  
**Technical quality:** not yet judgeable; method unimplemented  
**Clarity:** 8/10  
**Potential significance:** 7/10  
**Current recommendation:** **Weak Reject / Borderline until anchor pilot succeeds**  
**Confidence:** medium

### What would move toward Accept

1. A strong held-out composition gain over verified-query memory.
2. Evidence that semantic canonicalization—not just generic compression—creates the gain.
3. A real workload result.
4. Auditable semantic purity and low negative transfer.
5. A deliberately small architecture.

---

## 7. Review conclusion for refinement

The proposal should be refined to one sentence:

> **SemLibSQL automatically induces typed warehouse-specific semantic operators from semantically normalized verified SQL histories and tests whether this new learned language yields compositional generalization beyond episodic query memory.**

Delete from the first paper:

- DenoRepair;
- automatic semantic-view materialization;
- long-horizon drift governance;
- RL;
- multi-agent debate;
- proof-carrying SQL;
- broad active exploration.

Those ideas may become independent papers if the core abstraction result is positive.

**Internal review verdict:** `REVISE -> READY_FOR_PILOT`, not formal acceptance.