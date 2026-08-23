# SemLibSQL Pilot Runbook

> This is a research execution runbook, not implementation code.
> Run order is chosen to maximize information gained per engineering hour.

## Stage 0 — Corpus sanity

Before any abstraction method:

1. sample 30 records;
2. verify motif labels independently;
3. verify that each high-priority motif contains at least 2 materially different SQL realization families;
4. verify hard negatives exist;
5. verify no target motif is encoded directly in filenames or metadata visible to the learner.

**Stop** if the corpus is too easy syntactically.

---

## Stage 1 — Baseline floor

Run:

- lexical/token clustering;
- normalized-template grouping;
- AST similarity clustering;
- AST anti-unification;
- Stitch-like compression.

Record:

- pairwise hard-positive recall;
- hard-negative false merge rate;
- motif purity;
- support diversity across realization families.

This stage establishes how much can be achieved without SQL-specific semantics.

---

## Stage 2 — ReGAL-style threat baseline

Approximate the strongest generic program-abstraction competitor:

1. propose reusable program fragments/functions from the SQL/relational corpus;
2. allow execution-based validation/refinement;
3. do not provide gold motif labels;
4. give access to the same verification budget used by SemLibSQL where fair.

The purpose is not exact reproduction of ReGAL. It is to eliminate the easy reviewer objection that generic execution-validated refactoring is enough.

---

## Stage 3 — Semantic normalization only

Add:

- relational logical-plan normalization;
- schema grounding;
- grain inference/annotation;
- temporal role representation;
- conservative join semantics.

Do **not** add behavioral signatures yet.

Question:

> Does representation alone improve cross-realization grouping?

---

## Stage 4 — Formal/differential equivalence evidence

For candidate merges:

- call formal SQL equivalence tooling when supported;
- otherwise run discriminating generated database instances;
- retain UNKNOWN rather than force a decision.

Measure:

- additional true merges;
- prevented false merges;
- solver coverage;
- timeout/UNKNOWN rate;
- runtime cost.

---

## Stage 5 — Abstraction mining

Run the same simple abstraction miner over:

1. AST representation;
2. logical plan;
3. semantic plan;
4. semantic plan + equivalence evidence.

Keeping the miner fixed isolates the value of representation.

Primary plot:

```text
hard-positive recall / semantic purity
             vs
        false-merge rate
```

Secondary plot:

```text
cross-realization support
             vs
        library size
```

---

## Stage 6 — Gate A decision

### PASS

Continue only if semantic representation/evidence finds reusable abstractions that strong generic baselines systematically miss while controlling false merges.

### FAIL

Stop. Do not build library-aware generation.

Write a short failure report identifying whether:

- corpus did not contain enough hidden semantic recurrence;
- syntax baselines were already sufficient;
- formal/behavioral equivalence had poor coverage;
- semantic annotations were too manual;
- abstractions were too context-specific.

---

## Stage 7 — Held-out composition only after PASS

Construct the composition split described in `PILOT_DATASET_PROTOCOL.md`.

Systems:

- stateless base model;
- raw verified-query retrieval;
- matched-context retrieval;
- structured verified memory;
- syntactic learned library;
- ReGAL-style library;
- oracle semantic library;
- SemLibSQL.

The anchor comparison is:

> SemLibSQL vs strongest structured-memory/generic-library competitor.

---

## Stage 8 — Error taxonomy

For each wrong held-out composition, assign one first-failure category:

- primitive not discovered;
- wrong primitive parameterization;
- primitive scope violation;
- composition/planning failure;
- lowering/SQL realization failure;
- schema linking failure unrelated to library;
- retrieval baseline had near-duplicate leakage;
- benchmark ambiguity/data issue.

This prevents incorrectly attributing all accuracy differences to abstraction learning.

---

## Stage 9 — Required negative-transfer test

Include tasks unrelated to learned motifs.

Compare:

- library disabled;
- library optional/routed;
- library always exposed.

If exposing the library creates systematic semantic bias on unrelated queries, the method needs a routing/scoping mechanism before scaling.

---

## Stage 10 — End-of-pilot artifact set

The first real experimental run should eventually emit:

- `CORPUS_MANIFEST.json`
- `MOTIF_LABELS.csv`
- `BASELINE_RESULTS.csv`
- `CANONICALIZATION_RESULTS.csv`
- `ABSTRACTION_LIBRARY.json`
- `GATE_A_REPORT.md`
- if PASS: `COMPOSITION_RESULTS.csv`
- if PASS: `GATE_B_REPORT.md`

No paper claim should be updated until these artifacts exist.