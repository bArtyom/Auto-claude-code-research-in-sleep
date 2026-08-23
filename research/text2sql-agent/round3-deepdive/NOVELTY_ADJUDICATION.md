# Round 3 Novelty Adjudication

> **Date:** 2026-08-23  
> **Workflow role:** `novelty-check`  
> **Scope:** DreamSQL / SemLibSQL, Result2SQL / DenoRepair, AutoSemanticView / Semantic Promotion.  
> **Review status:** literature-grounded internal adjudication. The required independent Codex/manual reviewer backend is not exposed in this ChatGPT connector session, so **cross-model novelty acceptance is unavailable**. All proceed verdicts are provisional.

## 0. Decision standard

For each candidate:

1. extract the claims that would have to be novel;
2. identify the nearest method and setting prior art;
3. distinguish a new mechanism from an "apply X to SQL" story;
4. require a cheap experiment that can falsify the differentiator;
5. kill or narrow ideas whose novelty depends on ignoring direct prior work.

---

# 1. Candidate A — DreamSQL → SemLibSQL

## Proposed method

Learn a warehouse-specific library of **new typed semantic operators** by compressing/canonicalizing a corpus of verified SQL solutions, then use those operators compositionally on future Text2SQL tasks.

## Core claims

| Claim | Novelty | Closest prior | Required delta |
|---|---:|---|---|
| Persistent reusable SQL experience helps future queries | LOW | AgentSM; Crystallization; continual feedback memory | Not a paper claim |
| Agents can learn reusable executable skills | LOW | Voyager; SAGE; SkillFoundry; EvoSkills; HASP; SkillCraft | Not a paper claim |
| Programs can be compressed into learned libraries | LOW | DreamCoder; Stitch; LILO | Inspiration/baseline |
| SQL solutions can be canonicalized by **behavioral/semantic equivalence** before abstraction mining | MEDIUM–HIGH | equality saturation + SQL rewrites + library learning | Must be a real algorithm, not prompt paraphrasing |
| Induced warehouse-specific semantic operators beat episodic query memory on **held-out compositions** | HIGHER / empirical | AgentSM/Crystallization do reuse; program-learning literature does composition | Needs leakage-resistant motif-composition split |
| Induced operators correspond to stable business semantics rather than syntax fragments | MEDIUM–HIGH / empirical | semantic layers / GATE learn semantics but not this abstraction mechanism | Need purity/intervention/manual audit |

## Closest-prior-work matrix

| Prior work | Overlap | Key difference required for SemLibSQL |
|---|---|---|
| DreamCoder | automatic DSL/library growth from solved programs | SQL semantic canonicalization + enterprise Text2SQL composition |
| Stitch | corpus-guided abstraction learning | mine across semantically equivalent but syntactically different SQL |
| LILO | LLM synthesis + library compression + documentation | business-semantic typed operators, not generic code functions |
| Voyager/SAGE/recent skill libraries | persistent learned executable skills | domain-specific semantic operator induction under SQL equivalence |
| AgentSM | structured SQL-agent trace memory | induce new abstractions instead of retrieve traces |
| Crystallization in Text2SQL | verified corrected queries become reusable memory | new compositional language rather than stored query exemplars |
| GATE | execution validates reusable semantic groundings | higher-order parameterized program operators mined across solutions |
| semantic-layer agents | stable curated business concepts improve Text2SQL | library is *automatically induced from verified workload* |

## Overall assessment

**Novelty score: 7/10, conditional.**

**Recommendation: PROCEED WITH CAUTION.**

The idea survives only in the narrow formulation:

> **semantics-aware library induction + typed reusable operators + held-out compositional evaluation beyond strong memory/retrieval baselines.**

If semantic canonicalization is shallow or the evaluation is ordinary random split, novelty falls to roughly **4/10**.

### Reviewer-risk sentence

> "This is DreamCoder/Stitch applied to SQL, and recent agents already learn skill libraries."

The paper must make this attack false experimentally, not rhetorically.

---

# 2. Candidate B — Result2SQL → DenoRepair

## Proposed method

Repair complex executable-but-semantically-wrong SQL from a few output-level constraints, using provenance/causality to localize the responsible query region and optimizing for full semantic repair with low collateral damage.

## Core claims

| Claim | Novelty | Closest prior | Required delta |
|---|---:|---|---|
| User marks missing/unexpected tuples and query is refined | LOW / NOT NOVEL | why-not refinement; 2013 user-feedback query refinement | Explicitly concede prior art |
| Synthesize SQL from result examples | LOW | PATSQL, Sickle, PBE | Not a paper claim |
| Interactive Text2SQL correction | LOW | Speak to Your Parser; DIY; later feedback systems | Different feedback and repair objective |
| Sparse heterogeneous denotational constraints repair complex analytical SQL | MEDIUM | query refinement + PBE | joins/aggregates/windows/temporal semantics; partial constraints |
| Provenance/causality localizes LLM SQL structural errors | MEDIUM | provenance/causality/query explanations | need operator-localization algorithm and evaluation |
| Collateral-damage / hidden-generalization objective distinguishes semantic repair from tuple patching | MEDIUM–HIGH as formulation | classical refinement often targets observed answer constraints | must evaluate on unseen rows/perturbed DB instances |

## Overall assessment

**Naive Result2SQL novelty score: 2–3/10 — ABANDON.**

**DenoRepair novelty score: ~6/10 — PROCEED WITH CAUTION as a separate line.**

The strongest contribution is likely:

> a modern benchmark/formulation for sparse denotational repair of semantically wrong analytical SQL + a provenance-guided structural localization method.

This is not as clean a first paper as SemLibSQL because direct database prior art is closer.

### Reviewer-risk sentence

> "Expected/missing tuple query refinement predates LLMs by more than a decade; the new part is only an LLM in the loop."

The defense must be complex structural errors + heterogeneous sparse constraints + hidden generalization + evidence that provenance localization contributes beyond the LLM.

---

# 3. Candidate C — AutoSemanticView → Semantic Promotion

## Proposed method

Promote repeatedly validated learned semantic abstractions into persistent governed data assets when their expected future utility exceeds creation/maintenance/drift risk.

## Core claims

| Claim | Novelty | Closest prior | Required delta |
|---|---:|---|---|
| Semantic layer improves agentic Text2SQL | LOW | semantic-layer agents / industry systems | baseline only |
| Learn reusable semantics from execution | LOW–MEDIUM | GATE | baseline/precursor |
| Automatically create semantic views from existing analytics assets | LOW / product collision | 2026 semantic-view autopilot/discovery products | unsafe novelty claim |
| Materialize recurring workload structure based on future utility | LOW | materialized view selection / common-subexpression reuse / self-driving DBs | old systems problem |
| Promote *learned semantic operators* with uncertainty, governance, drift and dependency-aware lifecycle | MEDIUM | intersection less direct | needs upstream SemLibSQL plus long-horizon system evaluation |

## Overall assessment

**Original AutoSemanticView novelty score: 3–4/10 — KILL AS STANDALONE PAPER 1.**

**Semantic Promotion extension: 6–7/10 potential, but high engineering scope.**

**Recommendation: MERGE INTO FUTURE WORK / PAPER 2.**

Do not spend implementation effort here before proving that the automatically learned semantic operators are useful.

---

# 4. Comparative ranking after deep search

| Rank | Direction | Novelty | Clean falsification | Engineering risk | Decision |
|---:|---|---:|---:|---:|---|
| 1 | **SemLibSQL (DreamSQL)** | 7/10 conditional | High | Medium | **PROCEED** |
| 2 | **DenoRepair (Result2SQL)** | 6/10 | High | Medium | **KEEP AS SEPARATE LINE** |
| 3 | **Semantic Promotion** | 6–7/10 conditional | Medium | High | **DEFER** |
| — | naive Result2SQL | 2–3/10 | — | — | **KILL** |
| — | naive AutoSemanticView | 3–4/10 | — | — | **KILL/MERGE** |

---

# 5. Recommended first-paper thesis

The first paper should contain **one dominant contribution**:

> **SemLibSQL learns a warehouse-specific semantic program library from verified Text2SQL experience. Unlike query-memory systems that retrieve prior solutions, it canonicalizes behaviorally equivalent SQL programs, induces parameterized typed operators, and tests whether these operators enable held-out compositional generalization at lower inference cost.**

Explicitly reject from Paper 1:

- result-table correction interface;
- provenance-guided repair;
- automatic view/materialized-view creation;
- long-horizon semantic governance;
- RL training of the base LLM;
- a large multi-agent architecture.

Those are future or independent papers.

---

# 6. What must be true before implementation deserves expansion

Run a cheap pilot first. Proceed to a larger system only if:

1. semantic canonicalization merges SQL implementations that syntactic mining fails to align;
2. learned abstractions have auditable semantic purity;
3. they improve held-out motif composition beyond verified-query retrieval/AgentSM-like memory;
4. gain survives matched token/context budget;
5. negative transfer is low;
6. the improvement appears on at least one enterprise-like/open real workload, not only generated toy motifs.

If conditions 1–3 fail, **stop**. Do not rescue the idea by adding more agents, RL, or reviewers.

---

# 7. Formal review state

The repository's `novelty-check` contract calls for an independent external reviewer. That backend is not available through this connector session.

```yaml
review_independence: unavailable
acceptance_status: provisional
formal_novelty_gate: REVIEW_UNAVAILABLE
```

The literature-grounded recommendation is therefore a **research prioritization decision**, not cross-model acceptance.