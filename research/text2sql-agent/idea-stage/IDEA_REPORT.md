# Idea Discovery Report — Text2SQL × Long-Horizon Database Agents

**Date:** 2026-08-23  
**Pipeline:** research-lit → idea generation → novelty filtering → adversarial review → refinement → experiment design  
**Status:** **IDEA STAGE COMPLETE; EMPIRICAL WORK NOT STARTED**  
**Formal external-review receipt:** unavailable in this connector session; novelty/review verdicts remain provisional.

---

## Executive summary

After breadth-first ideation, several rounds of recent-literature search, and repeated elimination, the remaining research bet is a deliberately narrow one:

> **SemLibSQL-Γ: learn reusable SQL abstractions together with the warehouse-specific guards under which those abstractions are semantically valid, then test whether this guarded language supports unseen Text2SQL compositions beyond strong memory and generic program-library learning.**

The most important final change is that **library learning, e-graph library learning, conditional rewriting, precondition inference, and SQL equivalence are all prior art**. Therefore none can be claimed separately. The only defensible hypothesis left is an empirical/method intersection:

> Real warehouse SQL may contain a **Guarded Abstraction Gap**: useful recurring semantic operations that become reusable only after conditioning equivalence on local data/business contracts, and whose safe reuse requires carrying those conditions forward.

The next question is empirical. No SQL corpus has been executed and no pilot result is claimed here. Per the user instruction, this workflow stops at that boundary.

---

# 1. Final literature landscape

## 1.1 Text2SQL memory and execution-grounded reuse are crowded

Recent systems already establish persistent reuse:

- **AgentSM: Semantic Memory for Agentic Text-to-SQL** (2026, arXiv:2601.15709) stores prior execution traces as interpretable structured programs and reuses them to shorten later reasoning.
- **GATE: Bootstrapping Semantic Layer from Execution for Text-to-SQL** (2026, arXiv:2606.05634) validates grounding hypotheses through execution and accumulates supported groundings as reusable memory.
- Continual human-feedback Text2SQL work distills corrections into reusable domain knowledge.
- Multi-turn memory benchmarks in 2026 explicitly evaluate working, episodic, and semantic memory.

**Conclusion:** storing/retrieving previous SQL, traces, corrections, or semantic groundings is a baseline, not a contribution.

## 1.2 Semantic-layer and schema-side adaptation are also crowded

- A 2026 semantic-layer-mediated Text2SQL agent decouples intent from physical SQL through an explicit semantic model/IR.
- Snowflake Semantic View Autopilot mines query history and BI assets to generate governed semantic views.
- **The Case for Text-to-SQL Friendly Logical Database Design** (2026, arXiv:2606.03145) uses historical question-SQL pairs to guide semantics-preserving logical views/partitions/renamings.

**Conclusion:** automatic semantic-layer generation or workload-guided logical views cannot be Paper-1 novelty.

## 1.3 Generic library learning is a major collision

- **DreamCoder** learns reusable program libraries from solved programs.
- **Stitch** performs scalable corpus-guided library induction.
- **babble / LLMT** (POPL 2023, arXiv:2212.04596) explicitly performs **library learning modulo an equational theory** using e-graphs and anti-unification, so it already attacks syntactic variation via semantic equalities.
- **ReGAL** (ICML 2024) refactors programs into reusable abstractions and uses execution for validation/refinement.
- **E-Stitch** (EGRAPHS 2026) extends Stitch's top-down library search directly to e-graphs, combining efficient corpus-guided search with equivalence classes.

**Conclusion:** `SQL history → e-graph/canonicalization → reusable functions` is not novel.

## 1.4 Conditional equivalence and guard reasoning are also prior art

The final novelty audit found additional close mechanisms:

- Conditional term rewriting is classical.
- **Combining E-Graphs with Abstract Interpretation** (2022) uses program-property analysis to discharge conditions for conditional rewrites.
- **Colored E-Graphs** and the 2026 **Predicate E-Graphs with Symbolic Conditional Rewriting** explicitly represent equality under assumptions/Boolean predicates.
- **Alive-Infer** (PLDI 2017) automatically infers valid preconditions for program optimizations from positive/negative examples and can generalize optimization patterns.
- Other program-analysis work studies automatic precondition inference more broadly.

**Conclusion:** even `learn/reason about a rewrite precondition` is not enough as a novelty claim.

## 1.5 SQL equivalence under constraints is mature database theory

Database research has long studied equivalence/reformulation under integrity constraints. More recent tools such as U-semiring-based reasoning and **VeriEQL** handle increasingly rich SQL fragments and constraints; modern rewrite systems continue to expand the space of validated SQL transformations.

**Conclusion:** SemLibSQL must use equivalence/constraint tooling as a component or oracle. It cannot claim new SQL equivalence checking.

## 1.6 Clarification, active exploration, and generic agent loops are crowded

- TRUST-SQL/SDE-SQL occupy active schema exploration and sequential decision-making territory.
- PRACTIQ, AmbiSQL, and related work occupy ambiguity/clarification.
- MCTS/test-time scaling and multi-agent SQL repair are already substantial literatures.

**Conclusion:** do not rescue the project by adding another planner/critic/search loop.

---

# 2. The surviving research object: Guarded Abstraction Gap

Let a warehouse contract `Γ` contain high-confidence facts such as:

- primary/foreign keys;
- uniqueness and nullability;
- dbt tests/contracts;
- validated lineage/relationship facts;
- governed metric definitions;
- evidence-backed business/data invariants.

For SQL subprograms `P` and `Q`, the relevant relation is not necessarily global equivalence but:

`P ≡_Γ Q`

meaning they are interchangeable for the relevant observable behavior when the applicable part of `Γ` holds.

Examples:

- `COUNT(*)` and `COUNT(DISTINCT customer_id)` coincide only under an appropriate uniqueness/grain condition.
- two latest-record idioms coincide only under specified key/time/tie semantics.
- an inner-join formulation and a relationship-based abstraction coincide only if coverage/cardinality assumptions hold.
- a base-table computation and a semantic/dbt view coincide only if the view's contract is valid for that context.

### Guarded Abstraction Gap (GAG)

Define the empirical gap between:

1. abstractions recoverable under syntax or a **global/fixed** equational theory, and
2. abstractions recoverable only when equivalence is conditioned on **local warehouse contracts**, while maintaining low false-merge rate.

If this gap is negligible on realistic workloads, the SemLibSQL line should be killed even if an implementation can be built.

This turns the project from “another library learner” into a falsifiable scientific question about the structure of enterprise SQL experience.

---

# 3. Final proposed method: SemLibSQL-Γ

Each learned library item is a pair:

```text
(operator, guard)
```

rather than an unguarded macro.

Example:

```text
operator:
  latest_snapshot(relation, key, effective_time, as_of)

guard:
  key denotes the target business grain
  effective_time defines the intended validity ordering
  rows after as_of are excluded
  tie semantics are resolved

evidence:
  verified supporting programs
  equivalence/counterexample results
  constraint provenance
```

### Pipeline

1. collect verified `(question, context, SQL, evidence)` histories;
2. lower SQL to a typed relational plan;
3. assemble candidate contract facts with provenance classes (`DECLARED`, `VERIFIED`, `EMPIRICAL`, `HYPOTHESIS`);
4. propose candidate repeated subprogram families;
5. search for both a reusable abstraction and a compact sufficient guard explaining which instances are interchangeable;
6. use conservative equivalence evidence: safe rewrites, bounded/formal checking where possible, and adversarial/differential database instances;
7. retain `EQUIVALENT / NOT_EQUIVALENT / UNKNOWN`; do not force unknown pairs together;
8. expose an operator only when its guard is satisfied in the new context;
9. otherwise fall back to the base Text2SQL system.

### Critical refinement

The paper must compare against a **composed PL baseline**, not strawmen:

> E-Stitch/babble-style library learning over e-graphs + conditional/predicate reasoning + Alive-Infer-style precondition inference.

If that composed baseline matches the proposed joint system, the method novelty is gone. A positive paper then requires either a strong new empirical finding (the Guarded Abstraction Gap benchmark/analysis) or a demonstrable advantage from **jointly** optimizing abstraction, guard compactness, workload reuse, and downstream Text2SQL composition.

---

# 4. Ranked ideas after the final novelty audit

| Rank | Direction | Novelty (provisional) | Cost to falsify | Verdict |
|---|---|---:|---:|---|
| 1 | **SemLibSQL-Γ / Guarded Abstraction Gap** | **5.5–6/10** | Low–Medium | **RECOMMENDED AS HIGH-RISK, CHEAP-TO-KILL BET** |
| 2 | **TemporalSQL-Drift** — historical business-semantics benchmark | 6.5/10 | Low–Medium | **BEST BACKUP** |
| 3 | **DualSQL** — cross-task dual-control exploration | 7/10 | High | MOONSHOT BACKUP |
| 4 | DenoRepair — sparse denotational-feedback repair | 5/10 | Medium | DEPRIORITIZED |
| 5 | AutoSemanticView | 3/10 | — | ELIMINATED AS PAPER 1 |

SemLibSQL remains first not because it has the highest raw novelty score, but because it has the sharpest **48-hour-style falsification path** and the strongest route from a positive mechanism result to a method paper.

---

# 5. Closest work and exact delta

| Work | What it already does | What SemLibSQL-Γ would still need to demonstrate |
|---|---|---|
| DreamCoder / Stitch | reusable program libraries | warehouse-conditioned semantics and Text2SQL reuse |
| babble / LLMT | library learning modulo supplied theory | local/workload-specific guarded equivalence, guard generalization, downstream Text2SQL composition |
| **E-Stitch (2026)** | efficient library learning directly over e-graphs | same as above; must be a strong baseline |
| ReGAL | execution-validated reusable refactorings | no warehouse-contract/guard objective |
| Predicate/Colored E-Graphs | conditional equality under assumptions | no Text2SQL library induction/compositional evaluation |
| Alive-Infer | automatic precondition inference for optimizations | no joint warehouse abstraction learning from Text2SQL workload |
| U-semiring / VeriEQL | SQL equivalence under constraints | no reusable language induction |
| AgentSM | structured Text2SQL semantic memory | retrieves/reuses programs rather than inducing a guarded language |
| GATE | execution-grounded reusable semantic facts | grounding memory, not guarded program abstraction |
| Snowflake Semantic View Autopilot | workload/BI → governed semantic model | product semantic modeling, not the GAG/guarded-library hypothesis |

### Final novelty verdict

**PROCEED ONLY AS A FALSIFICATION BET.** A paper is not justified by implementation novelty alone. Gate A must reveal a meaningful Guarded Abstraction Gap on realistic SQL and show that it is not already captured by a strong composite PL baseline.

---

# 6. Claims and kill gates

## C1 — Existence of a Guarded Abstraction Gap

**Claim:** realistic Text2SQL histories contain high-value hard positives that global/fixed-theory library learning misses but warehouse-conditioned equivalence recovers at comparable false-merge precision.

**Kill if:** the strongest E-Stitch/babble/ReGAL composite recovers the same abstractions, or any gain is mostly alias/CTE/format normalization.

## C2 — Guard learning improves safety

**Claim:** inferred/derived applicability guards reject near-miss reuse cases where an unguarded library silently applies the wrong semantic operation.

**Kill if:** guard information adds negligible safety, or requires motif-by-motif manual labeling.

## C3 — Guarded abstractions beat memory on unseen composition

**Claim:** on withheld motif combinations, the learned guarded language outperforms matched-context verified-query and structured-memory baselines.

**Kill if:** memory closes the gap or gains are tiny/unstable/single-database.

## C4 — Joint induction matters (optional method claim)

**Claim:** jointly selecting abstraction + guard for downstream reuse beats a pipeline that independently applies E-Stitch/babble then Alive-Infer-style guard inference.

**Kill if:** the pipeline baseline is equivalent. In that case, retain only a benchmark/finding paper if the Guarded Abstraction Gap itself is strong.

---

# 7. Possible paper outcomes

| Gate-A finding | Gate-B finding | Honest outcome |
|---|---|---|
| no GAG | — | kill SemLibSQL line |
| GAG exists, generic composite baseline handles it | useful or not | benchmark/analysis paper at best; no strong method claim |
| GAG exists, joint method improves abstraction/safety | composition fails | analysis/method component paper; do not claim Text2SQL benefit |
| GAG exists, joint method improves and composition wins | positive | full SemLibSQL method paper |
| oracle guarded library helps, automatic guard/abstraction induction fails | mixed | automatic induction remains unsolved; negative diagnostic result |

---

# 8. Best backup: TemporalSQL-Drift

If Gate A kills SemLibSQL, the most defensible next idea is a benchmark around **business-semantic drift over time**, distinguishing:

1. the definition valid at historical event time;
2. what the agent knew at historical knowledge time;
3. recomputation of historical data under today's definition.

General bitemporal memory is not new, so the contribution would be a Text2SQL-specific evaluation of changing metric/entity semantics and historical reproducibility. The final literature search found substantial industry concern about metric-definition drift but no directly matching Text2SQL benchmark in the searched results.

---

# 9. Eliminated directions

- active schema exploration / POMDP as the main novelty;
- generic semantic/failure memory;
- generic semantic IR/compiler;
- naive result-feedback SQL repair;
- automatic semantic-view generation as Paper 1;
- generic ambiguity/clarification;
- MCTS/test-time scaling as novelty;
- memory poisoning as main Text2SQL thesis;
- generic program-library learning;
- generic e-graph/conditional rewrite/precondition inference.

These can appear only as components, baselines, or future extensions.

---

# 10. Canonical deliverables

- `idea-stage/IDEA_REPORT.md` — this canonical decision record.
- `idea-stage/IDEA_CANDIDATES.md` — compact shortlist.
- `idea-stage/docs/research_contract.md` — active research contract.
- `refine-logs/FINAL_PROPOSAL.md` — frozen method formulation.
- `refine-logs/EXPERIMENT_PLAN.md` — Gate A/B/C design.
- `refine-logs/EXPERIMENT_TRACKER.md` — all empirical runs currently NOT STARTED.

Historical breadth/deep-dive material remains under the earlier round directories and is not the active specification.

---

# 11. Stop boundary

The idea stage has reached the point where the remaining uncertainty is **empirical rather than conceptual**:

- Is the Guarded Abstraction Gap actually large enough to matter?
- Can guards be inferred/verified without hand-authoring the answer?
- Does any advantage survive E-Stitch/babble + conditional-equality + precondition-inference baselines?
- Does it help held-out Text2SQL composition beyond memory?

Answering those questions requires constructing executable SQL/database cases and running Gate A.

**No experiment has been run. No pilot signal is claimed.**

Per the user's requested boundary, stop here before empirical execution.

---

## Final idea-stage verdict

**Active bet:** SemLibSQL-Γ / Guarded Abstraction Gap  
**Status:** idea specification frozen; ready for empirical falsification  
**Novelty confidence:** moderate-low but scientifically testable; strongest collisions explicitly baselined  
**Formal independent reviewer gate:** unavailable / not passed  
**Next action when resources are provided:** run Gate A only; do not build the full agent first.
