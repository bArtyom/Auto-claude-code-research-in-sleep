# Round 4 — Novelty Recheck for SemLibSQL

> Workflow role: `novelty-check`
> Status: provisional; no independent cross-family reviewer backend is available in this connector session.

## 1. Core claims under scrutiny

### Claim A
Verified SQL histories can be converted into a learned reusable library rather than retrieved episodically.

**Novelty: LOW by itself.** DreamCoder, Stitch, babble, and ReGAL already establish reusable library induction from program corpora.

### Claim B
SQL-specific semantic canonicalization can collapse surface-different implementations into a common warehouse-semantic motif before abstraction mining.

**Novelty: MEDIUM.** SQL equivalence is well studied, and e-graph/equality-saturation machinery is not new. The potentially novel part is using a conservative hybrid equivalence relation tailored to **warehouse business motifs** as the substrate for automatic library induction.

### Claim C
The induced library improves **held-out motif composition** beyond strong structured verified-query memory.

**Novelty: MEDIUM-HIGH as an empirical finding.** Compositional Text2SQL benchmarks exist, but the proposed experiment distinguishes episodic query memory from a learned warehouse-specific language under a controlled workload-history split.

### Claim D
The automatically learned library approaches a human semantic-layer/operator library while remaining scoped and safe.

**Novelty: MEDIUM.** This is a useful evaluation axis rather than a standalone algorithmic claim.

---

## 2. Closest prior work

| Work | Overlap | Threat level | Required delta |
|---|---|---:|---|
| DreamCoder | learns reusable abstractions from solved programs | high | SQL/warehouse semantic canonicalization + Text2SQL task-stream evaluation |
| Stitch | fast corpus-guided library learning | high | show syntactic compression misses semantically equivalent SQL realizations |
| babble | e-graph + anti-unification abstraction learning | high | e-graphs cannot be claimed as novelty; business-semantic typed equivalence must matter |
| ReGAL (ICML 2024) | learns execution-validated reusable functions by refactoring programs | **very high** | prove SQL-specific semantic normalization materially beats generic execution-validated refactoring |
| AgentSM (2026) | reusable structured semantic memory for agentic Text2SQL | high | library induction must outperform retrieval/reuse of prior structured programs on unseen compositions |
| GATE (2026) | execution-grounded reusable semantic memory | high | induced operators must abstract across multiple grounded memories rather than store individual groundings |
| SQL workload mining | recurring SQL patterns/templates | medium | promote patterns into typed operators and test downstream generative composition |
| Cosette / SQLSolver | SQL equivalence proving | medium | use as verification substrate; do not claim equivalence checking itself |
| Spider-CG / CoSQL-CG | compositional Text2SQL evaluation | medium | history-conditioned warehouse-specific induced-language split |

---

## 3. What must be removed from the paper pitch

Do **not** pitch any of the following as the main novelty:

- "we learn reusable functions from past SQL";
- "we use e-graphs to find common abstractions";
- "we verify abstractions by execution";
- "we canonicalize equivalent SQL";
- "we test compositional generalization";
- "we use semantic memory."

All of these have close prior art.

---

## 4. Narrowed paper claim

The defensible thesis is:

> **Generic program-library learners are biased toward repeated program form. Enterprise SQL contains repeated business semantics realized through heterogeneous relational implementations. A conservative SQL/warehouse-semantic canonicalizer exposes those latent repeated motifs, enabling automatic induction of typed operators that improve unseen motif composition beyond structured verified-query memory.**

This is both more specific and more falsifiable.

---

## 5. Reviewer attack simulation

### Attack 1 — "This is ReGAL on SQL"

Response required by evidence, not rhetoric:

- run ReGAL/Stitch-like baselines on the same corpus;
- construct motif families with intentionally different SQL realizations;
- show semantic canonicalization recovers cross-realization support that generic refactoring misses;
- show downstream held-out composition gain tracks those recovered abstractions.

If these tests fail, the attack is correct and the project should be killed or reframed as a benchmark/finding paper.

### Attack 2 — "This is just a semantic layer learned from query logs"

Response:

- semantic-layer products/work already infer/curate semantic assets;
- the paper therefore centers on **automatic operator induction + compositional generalization**, not semantic-layer creation;
- no claim that materializing/publishing semantic views is new.

### Attack 3 — "Retrieval with enough context can do the same thing"

Response:

- matched-context retrieval baseline;
- structured memory baseline;
- no near-duplicate full target query in history;
- report nearest-history similarity;
- force novel motif combinations.

If matched-context memory closes the gap, the language-learning claim fails.

### Attack 4 — "Your semantic canonicalizer is a pile of hand-written rules"

Response:

- report exactly which rules are generic relational equivalences vs warehouse annotations;
- ablate each layer;
- quantify coverage, UNKNOWN rate, and false-merge rate;
- compare with formal equivalence tools where possible;
- keep claims about partial conservative equivalence only.

### Attack 5 — "The learned primitives are just human labels hidden in annotations"

Response:

- create a minimal-annotation track using only schema + verified SQL + execution;
- separately evaluate optional semantic metadata;
- report how much gain comes from annotations versus structural/behavioral evidence.

---

## 6. Novelty score after Round 4

- Broad DreamSQL concept: **4/10** — too close to general library learning.
- SemLibSQL as currently narrowed: **7/10 provisional**.
- SemLibSQL without a strong ReGAL/Stitch comparison: **3/10**.
- SemLibSQL with a positive cross-realization canonicalization result and held-out composition gain: **8/10 potential**.

## 7. Recommendation

**PROCEED ONLY TO THE CHEAP PILOT.**

No large agent implementation, training run, or semantic-view machinery should be built before the pilot answers:

> Does SQL-specific semantic canonicalization recover reusable abstractions that generic program-library learning misses?