# Idea Discovery Report — Text2SQL × Long-Horizon Database Agents

**Direction:** Text2SQL, SQL agents, and long-lived database agents  
**Date:** 2026-08-23  
**Pipeline:** research-lit → idea generation → novelty check → adversarial research review → method refinement → experiment planning  
**Status:** **IDEA DESIGN COMPLETE; EMPIRICAL VALIDATION NOT STARTED**  
**Formal cross-model review receipt:** unavailable in this connector session; all novelty/review verdicts below are provisional until a configured independent reviewer returns an identity-bearing verdict.

---

## 1. Executive summary

After several breadth-first rounds and repeated literature-driven elimination, the recommended research direction is:

> **SemLibSQL-Γ: Contract-Conditioned Semantic Library Learning from Verified Text-to-SQL Experience.**

The research question is no longer whether a Text2SQL agent can store previous queries, retrieve previous reasoning traces, construct a semantic layer, or learn a generic library of repeated code. All of those neighborhoods now have strong prior art.

The remaining hypothesis is narrower:

> **Can a long-lived Text2SQL agent discover reusable warehouse-specific semantic operators by grouping SQL programs that are equivalent only under a warehouse's integrity/business contract, attach the required scope conditions to each operator, and then use those scoped operators to solve unseen compositions better than strong memory and generic program-refactoring baselines?**

This is intentionally a sharp, falsifiable claim. The next required action is a controlled Gate-A experiment. Per the user's instruction, this research run stops before executing that experiment.

---

# 2. Literature landscape after final re-check

## 2.1 Text2SQL memory is crowded

Recent work makes ordinary persistent-query memory an insufficient novelty claim:

- **AgentSM: Semantic Memory for Agentic Text-to-SQL** (2026, arXiv:2601.15709) stores prior execution traces as structured programs and reuses them to shorten later reasoning trajectories.
- **GATE: Bootstrapping Semantic Layer from Execution for Text-to-SQL** (2026, arXiv:2606.05634) keeps grounding hypotheses open, uses execution feedback to validate them, and stores validated groundings as reusable memory.
- **Continual Learning of Domain Knowledge from Human Feedback in Text-to-SQL** (2025, arXiv:2511.10674) distills human feedback into structured reusable domain knowledge.
- **EvoSQL** (2026, arXiv:2607.20489) adds contextualized candidate memory to a critic-generator co-evolution framework.

**Implication:** `store/retrieve prior SQL`, `store successful traces`, `store failure tips`, and `execution-grounded semantic memory` are baselines, not the paper contribution.

## 2.2 Generic program-library learning is also strong prior art

The closest program-synthesis literature is more dangerous than initially assumed:

- **DreamCoder** learns reusable program libraries in a wake/sleep synthesis loop.
- **Stitch / Top-Down Synthesis for Library Learning** (arXiv:2211.16605) performs corpus-guided abstraction learning at much larger scale than earlier deductive approaches.
- **babble: Learning Better Abstractions with E-Graphs and Anti-Unification** (arXiv:2212.04596) introduces **library learning modulo theory (LLMT)**: abstractions are learned while an equational theory and e-graphs collapse syntactic variation.
- **ReGAL: Refactoring Programs to Discover Generalizable Abstractions** (ICML 2024) learns reusable functions from existing programs and validates/refines abstractions through execution.

**Critical consequence:** a paper whose contribution is simply `SQL history → canonicalize → mine reusable functions` is not sufficiently novel. In particular, babble already establishes the general idea of library learning modulo an equivalence theory.

## 2.3 SQL equivalence under constraints is a mature database topic

The database literature also removes several possible novelty claims:

- U-semiring work formalizes semantic equivalence for sophisticated SQL and integrity constraints.
- **VeriEQL** (2024, arXiv:2403.03193) performs bounded equivalence checking for complex SQL under integrity constraints and can produce counterexamples.
- Query reformulation under integrity constraints is decades old; equivalence under constraints is explicitly weaker than global equivalence.
- Recent work such as **SLER** (2026, arXiv:2603.04169) automatically builds very large SQL rewrite-rule repositories.

**Implication:** SemLibSQL must not claim new SQL equivalence checking, new constraint-aware query rewriting, or new rewrite-rule discovery.

## 2.4 Automatic semantic-layer generation is rapidly becoming productized

This area is especially crowded in 2026:

- A semantic-layer-mediated NL2SQL agent (arXiv:2606.31041) reasons over a curated Semantic Model Query IR and compiles it to multiple dialects.
- Snowflake **Semantic View Autopilot** mines query history and BI assets to generate governed semantic views; Snowflake engineering has publicly described query-history mining for semantic-model construction.
- **The Case for Text-to-SQL Friendly Logical Database Design** (arXiv:2606.03145) explicitly studies semantics-preserving schema abstraction, including logical views that materialize recurring join paths, guided by historical question-SQL pairs.

**Implication:** the earlier AutoSemanticView idea is not suitable as the first paper. Environment modification may remain a downstream extension, but not the current novelty anchor.

## 2.5 Clarification/ambiguity is also crowded

- **PRACTIQ** (NAACL 2025) benchmarks ambiguous and unanswerable conversational Text2SQL.
- **AmbiSQL** (2025) performs interactive ambiguity detection and resolution.
- **CLUES** (2026, arXiv:2602.12015) explicitly separates input ambiguity from model instability in a clinical Text2SQL case study.

**Implication:** generic `candidate disagreement → ask user` or `measure persistent ambiguity` is no longer a strong standalone contribution.

---

# 3. Ranked ideas after novelty filtering

| Rank | Idea | Novelty assessment | Feasibility | Main risk | Status |
|---|---|---:|---:|---|---|
| 1 | **SemLibSQL-Γ: contract-conditioned semantic library learning** | 7/10 provisional | Medium | could be judged as babble/ReGAL + SQL constraints | **RECOMMENDED** |
| 2 | **TemporalSQL-Drift: bitemporal business-semantics benchmark** | 6.5/10 | High | general bitemporal agent-memory work already exists | BACKUP |
| 3 | **DualSQL: cross-task dual-control database exploration** | 7/10 | Medium-Low | heavy sequential/RL evaluation; TRUST-SQL/SDE-SQL nearby | BACKUP / MOONSHOT |
| 4 | DenoRepair: sparse result-feedback repair for analytical SQL | 5/10 | High | result-feedback query refinement exists since at least 2013 | DEPRIORITIZED |
| 5 | AutoSemanticView / self-materializing semantic layer | 3/10 | Medium | Snowflake Autopilot + recent logical-design paper collide directly | ELIMINATED AS PAPER 1 |
| 6 | Persistent ambiguity / disagreement persistence | 4/10 | High | PRACTIQ, AmbiSQL, CLUES substantially occupy space | ELIMINATED AS MAIN THESIS |
| 7 | Basic failure/semantic memory | 2/10 | High | AgentSM, GATE, continual-feedback Text2SQL | ELIMINATED |

The recommendation optimizes for **one sharp mechanism with a cheap kill test**, rather than highest speculative novelty.

---

# 4. Recommended idea — SemLibSQL-Γ

## 4.1 Problem anchor

A long-lived SQL agent repeatedly sees different implementations of the same hidden warehouse operation:

- latest valid record as of a date;
- slowly-changing-dimension state lookup;
- deduplicate before a many-to-one join;
- fiscal-period mapping;
- recognized-revenue calculation;
- eligible-population denominator;
- bridge-table traversal.

Nearest-query memory can reuse old answers, while generic library learners can compress recurring syntax. However, SQL has a distinctive form of reuse:

> Two programs can be equivalent **only because a particular warehouse satisfies specific constraints or business contracts**.

Examples:

- `COUNT(*)` and `COUNT(DISTINCT id)` are equivalent only under a uniqueness condition on `id`.
- an inner join and a semijoin-like formulation can coincide under key/existence constraints;
- two latest-record idioms can be interchangeable only under timestamp/key tie assumptions;
- a physical table, a dbt model, and a semantic view may be interchangeable only under an asserted data contract.

The relevant object is therefore **contextual equivalence**, not global syntax equality.

## 4.2 Formal target

Let `Γ` denote a warehouse contract containing supported facts such as:

- primary / foreign keys;
- uniqueness and nullability;
- dbt tests and declared contracts;
- validated lineage relationships;
- verified business rules;
- evidence-backed data invariants.

For programs `P` and `Q`, define the target relation:

`P ≡_Γ Q` iff, for databases satisfying the relevant contract `Γ`, the programs have the same intended observable behavior.

SemLibSQL-Γ does **not** need to prove full SQL equivalence. It maintains a conservative three-valued judgment:

`EQUIVALENT / NOT_EQUIVALENT / UNKNOWN`.

## 4.3 Key method idea

For each reusable operator, learn not only a body but also a **scope contract**:

```text
latest_snapshot(relation, key, effective_time, as_of)
requires:
  key identifies one business entity
  effective_time defines valid ordering
  tie behavior is specified
  rows after as_of are excluded
```

The core research object is therefore:

> **abstraction + applicability conditions + evidence**, rather than abstraction alone.

A proposed pipeline:

1. collect verified `(question, warehouse context, SQL, execution evidence)` histories;
2. lower SQL into a relational/typed plan;
3. assemble candidate warehouse contracts from explicit metadata plus evidence-backed inferred constraints;
4. construct hard-positive and hard-negative candidate pairs/subprograms;
5. apply constraint-aware equivalence evidence: formal/bounded reasoning where supported, plus contract-preserving counterexample/differential execution otherwise;
6. induce reusable parameterized abstractions over equivalence-supported instances;
7. infer a compact sufficient scope contract for each abstraction rather than assuming universal validity;
8. expose only in-scope abstractions to later Text2SQL generation;
9. fall back to the base generator whenever applicability is unknown.

## 4.4 What is actually novel if the idea succeeds

The defensible contribution is **not** any individual component. It is the empirical/method claim that:

1. fixed-theory/generic library learning misses valuable SQL reuse because useful equivalences are **warehouse-conditional**;
2. learning/scoping abstractions with explicit applicability contracts recovers that reuse without unsafe false merges;
3. those scoped abstractions support unseen composition better than episodic/structured memory.

This is the exact claim the first experiment must attempt to falsify.

---

# 5. Closest prior work and delta

| Prior work | Strong overlap | Remaining delta SemLibSQL-Γ must prove |
|---|---|---|
| DreamCoder / Stitch | library induction from program corpora | SQL/warehouse contextual equivalence and scoped applicability |
| **babble / LLMT** | library learning modulo an equational theory; handles syntactic variation with e-graphs | theory/validity is not assumed globally fixed; abstractions carry warehouse-specific conditions and are evaluated for Text2SQL composition |
| **ReGAL** | execution-validated refactoring into reusable functions | no contract-conditioned query equivalence or SQL-specific scope safety objective |
| U-semiring / VeriEQL | SQL equivalence under integrity constraints | no program-library induction or held-out Text2SQL composition |
| AgentSM | reusable structured semantic memory | retrieve prior structured programs rather than induce a new reusable language |
| GATE | execution-grounded reusable semantic memory | stores validated grounding facts, not abstraction + precondition induction |
| Snowflake Semantic View Autopilot | mines workload/BI assets into semantic models | product-level governed semantic model generation; does not establish contract-conditioned program-library learning versus PL baselines |
| Text-to-SQL Friendly Logical Database Design | workload-guided schema abstractions/views | changes logical schema; does not learn scoped program abstractions for unseen composition |

### Novelty verdict

**PROCEED WITH CAUTION.** The intersection remains plausible, but the collision with **babble** is serious. If Gate A shows that a strong babble/ReGAL-style baseline with a carefully supplied SQL theory matches SemLibSQL-Γ, the idea should be killed rather than rescued with additional agent machinery.

---

# 6. Core claims and kill conditions

## C1 — Contract-conditioned abstraction discovery

**Claim:** warehouse-conditioned equivalence improves hard-positive abstraction recall at matched false-merge precision relative to syntactic/global-theory library learning.

**Kill if:** a strong fixed-theory babble/Stitch/ReGAL-style baseline is statistically/qualitatively equivalent after receiving safe SQL normalization rules.

## C2 — Scoped abstractions generalize compositionally beyond memory

**Claim:** automatically induced scoped operators improve unseen motif compositions over the strongest verified-query / structured-memory baseline at matched context budget.

**Kill if:** memory closes the gap, or gains are tiny/unstable/single-database.

## C3 — Scope contracts are necessary for safety

**Claim:** attaching applicability conditions reduces negative transfer on near-miss/hard-negative cases compared with an unscoped learned library.

**Kill if:** scope information provides no measurable safety benefit or is effectively supplied by hand.

---

# 7. Internal adversarial review

Because a configured independent reviewer backend is not exposed in this session, this is **same-model red-team analysis, not external acceptance**.

### Strongest reviewer objection 1: “This is just babble applied to SQL.”

This objection is valid unless the experiment isolates **context-conditioned applicability**. The paper must directly compare against LLMT/babble-style library learning with a generous SQL equational theory.

### Strongest reviewer objection 2: “Integrity-constraint query equivalence is old database theory.”

Also valid. Equivalence machinery must be treated as an oracle/component. The contribution must be about learning scoped reusable abstractions for Text2SQL, not equivalence itself.

### Strongest reviewer objection 3: “Snowflake already mines query logs into semantic views.”

This prevents broad product claims about automatic semantic-layer construction. The paper must focus on the narrower scientific hypothesis: program abstraction under contextual equivalence and held-out composition versus memory/library-learning baselines.

### Strongest reviewer objection 4: “A human semantic layer will obviously win.”

Include an oracle/manual semantic library as an upper bound. If automatic induction recovers little of its benefit, report failure rather than overclaiming.

### Strongest reviewer objection 5: “The benchmark can be gamed by templates.”

The held-out split must prevent full-query near duplicates, literal-only paraphrases, and single-schema memorization. Hard positives must contain genuinely distinct implementation families; hard negatives must differ on decision-relevant semantics.

---

# 8. Backup ideas

## 8.1 TemporalSQL-Drift — backup with lower implementation risk

Build a benchmark where warehouse/business definitions change over time and questions distinguish:

- semantics valid at historical event time;
- semantics known by the agent at historical knowledge time;
- recomputation of historical data under today's definition.

General bitemporal agent-memory work already exists, including recent 2026 work, so the contribution must be Text2SQL-specific **business-semantic drift evaluation**, not “bitemporal memory” itself.

Use this backup if SemLibSQL Gate A fails but the project still wants a long-horizon database-agent paper.

## 8.2 DualSQL — higher-risk backup

Optimize database probes for both current-query success and reduction of uncertainty for future tasks in the same warehouse.

This differs from SDE-SQL/TRUST-SQL only if evaluation is truly cross-task and cumulative. It is scientifically attractive but requires a longer sequential environment and is therefore not the first experiment to run.

---

# 9. Eliminated ideas and why

- **Generic active schema/POMDP agent** — TRUST-SQL and SDE-SQL make this crowded.
- **Basic semantic/failure memory** — AgentSM, GATE, continual human-feedback Text2SQL.
- **Generic semantic IR/compiler** — semantic-layer-mediated agents already provide this.
- **Naive Result2SQL** — *A framework for query refinement with user feedback* (2013) already refines SQL from unexpected/missing result tuples; provenance/why-not literature is even older.
- **AutoSemanticView as Paper 1** — Snowflake Semantic View Autopilot plus recent workload-guided logical-design work collide directly.
- **Generic ambiguity/clarification** — PRACTIQ, AmbiSQL, CLUES.
- **MCTS/search as novelty** — Alpha-SQL and extensive test-time-scaling work already occupy it.
- **Memory poisoning as main direction** — 2026 general agent-memory security has rapidly become crowded (MemSecBench, MemPoison, sleeper poisoning, etc.).

---

# 10. Refined deliverables

- Active research contract: `research/text2sql-agent/idea-stage/docs/research_contract.md`
- Refined proposal: `research/text2sql-agent/refine-logs/FINAL_PROPOSAL.md`
- Experiment plan: `research/text2sql-agent/refine-logs/EXPERIMENT_PLAN.md`
- Experiment tracker: `research/text2sql-agent/refine-logs/EXPERIMENT_TRACKER.md`
- Historical exploratory material: `research/text2sql-agent/round1-*` through `round4-semlibsql-pilot/` and the earlier top-level research memos.

---

# 11. Stop boundary: experiment is now required

The idea stage can no longer improve materially by adding more agent components. The next unresolved questions are empirical:

1. Do warehouse-conditioned equivalence judgments actually recover hard-positive motif families that strong babble/ReGAL-style baselines miss?
2. Can the method keep false semantic merges low on near-miss queries?
3. Does the induced library help held-out composition beyond strong verified memory?

Answering these requires building/executing the controlled SQL corpus described in the experiment plan. **No experiment has been run in this idea-discovery phase.**

Per the user instruction, stop here rather than fabricate pilot evidence.

---

# 12. Idea-stage verdict

**Selected:** SemLibSQL-Γ / Contract-Conditioned Semantic Library Learning  
**Novelty:** promising but collision-sensitive; provisional 7/10  
**Method maturity:** sufficiently specified for a falsification pilot  
**Empirical evidence:** none yet  
**Formal external-review gate:** unavailable / not passed  
**Next action:** run Gate A only; continue to Gate B only if Gate A survives.
