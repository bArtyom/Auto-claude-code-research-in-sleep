# Database-Venue Anchor Failure Ideas — Text2SQL × Data Agents

**Date:** 2026-08-26  
**Branch:** `research/text2sql-agent-fresh-2026-08-24`  
**Method:** strong database paper → evidence card → one hidden assumption → 20–50 example falsification → method only if failure reproduces  
**Empirical pilots:** NOT RUN  
**Formal cross-model review:** UNAVAILABLE in this connector session; novelty is provisional.

---

## 0. Why this second anchor batch exists

The first anchor batch focused on BIRD-INTERACT, SQL-Trail, DPC, VET, Spider 2.0 and MTSQL-R1. This batch deliberately shifts toward SIGMOD/VLDB work so the derived ideas are grounded in database-system assumptions rather than another LLM orchestration pattern.

The selection rule is strict:

1. prefer accepted 2026 SIGMOD/VLDB/PVLDB work;
2. prefer public code/data or a clearly reproducible artifact;
3. do not turn the paper's method name into the new idea;
4. isolate a property that the anchor does **not** directly evaluate;
5. define a cheap test before defining a repair mechanism;
6. if an old database literature already owns the mechanism, treat it as a baseline and move the novelty to the newly observed failure boundary.

The anchors are:

- **Schema Subsetting / BigBird / SKALPEL** — VLDB 2026 EA&B.
- **OpenSQL** — PVLDB 2026; public code.
- **GLEAM** — SIGMOD 2026; public code + seven real-world datasets.
- **SemBench** — VLDB 2026; public benchmark code/data and repeat-experiment harness.
- **ReSequel** — VLDB 2026; public code + eight workloads × three DBMSs.
- **SafeQL** — VLDB 2026; search-based refinement with DBMS-guided safe query space.
- **SQL-Factory** — PVLDB/VLDB 2026; public code and 300K+ generated SQL corpus.

---

# 1. Anchor evidence cards

## B1 — Large-Schema Schema Subsetting / BigBird / SKALPEL (VLDB 2026)

**Core question.** Does pruning the schema help or hurt NL-to-SQL when schemas become large?

**Evidence.** The VLDB 2026 experiment/analysis paper evaluates seven real-world schema-subsetting modules across BIRD, Spider 2 and SNAILS, introduces BigBird for larger-schema evaluation, and reports a nontrivial regime change: on smaller schemas, many subsetting methods reduce execution accuracy by roughly 3–10 percentage points; on large schemas, some methods improve accuracy by roughly 2–7 points or save tokens while maintaining accuracy.

**Why this is a useful anchor.** It already kills the simplistic claim `schema pruning is always good`. The unresolved scientific question is *what structural property causes the regime change*.

**Artifact status.** Paper is accepted at VLDB 2026. The authors' lab page states code/data for the SKALPEL project are coming soon; therefore any immediate reproduction that requires unreleased BigBird artifacts is **blocked**, not silently replaced by synthetic data.

**Do not propose.** Another generic schema linker, another embedding retriever, or `adaptive top-k schema selection` without first isolating the failure structure.

## B2 — OpenSQL (PVLDB 2026)

**Core hypothesis.** Sparse `(question, SQL)` supervision becomes much more useful when expanded into task-specific intermediate targets: global/local schema linking, diversified SQL generation and stepwise SQL selection.

**Evidence.** OpenSQL reports 70.0% BIRD-dev execution accuracy from only 14K annotated pairs with a 32B backbone, versus the much larger training corpus used by OmniSQL. The paper explicitly uses a synthetic augmentation pipeline for multiple intermediate learning tasks and releases preprocessing, augmentation, training and inference code.

**Explicit boundaries.** The paper notes: (i) full-schema global linking can fail on extremely wide schemas; (ii) generated explicit reasoning/CTE structures may increase query execution time; and (iii) flawed intermediate reasoning can still exist despite automated synthetic supervision.

**Hidden assumption worth testing.** Intermediate supervision increases label density, but if the root `(question, SQL)` pair is wrong or semantically incomplete, the same error may be **fanned out into several correlated training targets**.

**Do not propose.** Generic synthetic CoT supervision, another selector, or another local/global linker.

## B3 — GLEAM: Generalized Entity Matching with Adaptivity via LLMs (SIGMOD 2026)

**Core hypothesis.** Heterogeneous entity matching can be made label-efficient through structure/content-aware blocking, adaptive thresholding with online feedback, and hierarchical LLM reasoning.

**Evidence.** GLEAM evaluates on seven heterogeneous real-world datasets from Machamp, publishes code/data, reports strong end-to-end F1, and reports up to 25.7% F1 improvement while reducing unnecessary LLM calls.

**Hidden assumption worth testing.** Entity-matching F1 treats matching errors approximately uniformly. Downstream analytics do not: one false match on a high-degree/high-value entity can bias an aggregate more than many harmless pair errors.

**Do not propose.** Another entity matcher or generic dirty-key join agent. The new object must be **downstream analytical consequence**, not pairwise matching accuracy.

## B4 — SemBench (VLDB 2026)

**Core hypothesis.** LLM-powered semantic query engines should be evaluated across modalities, operators, scenarios, quality, cost, latency and memory rather than by isolated model accuracy.

**Evidence.** SemBench is public and supports LOTUS, Palimpzest, ThalamusDB, BigQuery and additional systems through a common runner. It contains 55 queries over table/text/image/audio data and semantic filter/join/map/rank/classify operators. The repository explicitly provides repeated-experiment scripts for error bars.

**Hidden assumption worth testing.** Reporting mean quality with error bars may still hide whether a user-visible **decision** (top-1 entity, top-k set, thresholded alert, final ranking) flips between repeated executions of the same stochastic semantic query.

**Do not propose.** Another semantic-query benchmark from scratch. Use SemBench itself to first establish whether repeatability/composition is a real failure.

## B5 — ReSequel (VLDB 2026)

**Core hypothesis.** LLM-generated rewrite rules can be made practical by query templatization, metadata guidance, sampled-data correctness verification and runtime ranking.

**Evidence.** ReSequel publishes a full reproduction repository with eight workloads, PostgreSQL/MySQL/DuckDB support, correctness-verification code and baseline integrations including SQLSolver. It reports workload speedups up to 16× over native DBMSs and 22× over LLM-based rewriting systems, with individual rewrites above 600×.

**Critical boundary.** The method verifies candidate rewrites for correctness on **sampled data**. Sample agreement is empirical evidence, not a proof of semantic equivalence. Rare values, NULL behavior, skew, boundary predicates and data correlations are exactly where an incorrect rewrite can escape sampling.

**Do not propose.** `Use SQLSolver instead` as the paper. Formal equivalence checking and test generation are prior art. First measure the real unsoundness rate and characterize when sampling fails.

## B6 — SafeQL (VLDB 2026)

**Core hypothesis.** Query repair should be a guided search in a safe query space, using DBMS feedback to repair erroneous components instead of regenerating an entire SQL statement.

**Evidence.** SafeQL targets invalid schema/function/value references and reports better execution accuracy/efficiency than regeneration-based repair on BIRD/Spider.

**Critical boundary.** DBMS errors are informative for non-executable SQL. Executable-but-semantically-wrong SQL often emits **no negative DBMS feedback at all**. The safe search may therefore have a sharp observability boundary: it can safely repair what the DBMS complains about but may be inert on silent semantic failures.

**Do not propose.** Generic semantic verifier or another repair loop. First quantify this observability boundary on public executable-wrong cases.

## B7 — SQL-Factory (PVLDB/VLDB 2026)

**Core hypothesis.** A multi-agent generation/expansion/management pipeline can synthesize a large, diverse, low-cost SQL corpus whose downstream use improves intelligent-database tasks.

**Evidence.** Public code; more than 300K executable queries generated for less than $200 API cost; paper emphasizes schema coverage and SQL diversity and reports downstream gains.

**Hidden assumption worth testing.** Corpus-level structural diversity is only a proxy for **marginal learning value**. Two corpora can have similar skeleton/schema diversity while differing dramatically in whether they cover the semantic failure modes that downstream Text2SQL models actually need to learn.

**Do not propose.** Another multi-agent synthetic SQL generator. First test whether current diversity/coverage metrics predict downstream error coverage.

---

# 2. Failure-derived idea bank

## Cluster 1 — Schema subsetting: what exactly gets lost?

### 1. JoinBridge Blind Spot

**Anchor:** VLDB 2026 schema-subsetting analysis.

**Failure boundary.** A schema subset can retain every lexically salient table/column in the question while dropping a low-salience bridge table, join key, time dimension or relationship needed to make the intended query valid.

**Hypothesis.** Catastrophic subsetting errors are disproportionately caused by missing **relational bridge context**, not missing directly mentioned attributes.

**Cheap test.** When the BigBird/SKALPEL artifacts are public, take 30–50 failed subsetting cases. Label omitted elements as direct target, filter/value support, join bridge/key, temporal dimension, or irrelevant. Compare the error distribution against random omitted schema elements and successful cases.

**Kill if.** Bridge omissions are not enriched among failures, or ordinary column recall predicts downstream success equally well.

**If positive.** A paper can first establish `relational-support recall` as the missing evaluation object. Only then consider a bridge-aware retrieval method.

### 2. Schema-Ambiguity Phase Transition

**Anchor:** same paper's finding that subsetting hurts small schemas but can help large ones.

**Failure boundary.** Raw schema size may not be the causal variable. A 500-column schema with distinctive names may be easier than a 100-column schema containing many semantically similar decoys.

**Hypothesis.** The sign of subsetting's benefit is better predicted by **ambiguity density / semantic decoy density** than by schema size alone.

**Cheap test.** Use existing natural BIRD/Spider2/SNAILS schemas only. Compute simple lexical/embedding neighborhood density around gold tables/columns and regress the full-schema-vs-subset accuracy delta against size versus ambiguity density.

**Kill if.** Raw schema size explains nearly all of the regime change, or ambiguity density adds no stable predictive value.

**No synthetic-core rule.** Artificially add decoy columns only after the natural-data signal exists.

### 3. Irreversible Pruning Boundary

**Failure boundary.** A schema subset error may become unrecoverable if downstream generation/execution tools are only allowed to reason inside the pruned schema.

**Hypothesis.** A substantial fraction of failures attributed to SQL reasoning are actually **information-set failures**: once the required element is pruned, later agent turns cannot recover it.

**Cheap test.** On naturally failed subsetting examples, compare the same generator under: fixed subset; subset + one-shot ability to reopen full schema; oracle-added missing element. Measure how much error disappears without changing the model.

**Kill if.** Reopening schema rarely fixes the error.

**Paper shape.** Boundary/diagnostic result; not a router paper unless the causal effect is large.

---

## Cluster 2 — Intermediate supervision can amplify bad labels

### 4. SupervisionFanout-SQL

**Anchor:** OpenSQL.

**Failure boundary.** A single erroneous or ambiguous `(question, SQL)` pair is transformed into multiple schema-linking, generation and selector supervision artifacts.

**Hypothesis.** Task decomposition creates a **label-error multiplier**: one bad gold example creates several mutually consistent but wrong targets, making the error more learnable than in ordinary end-to-end SFT.

**Cheap test.** Use 20–50 publicly audited BIRD examples with known gold/annotation problems. Run OpenSQL's data-augmentation pipeline without training. For each bad root pair, count which generated intermediate labels inherit the error and whether any stage detects/neutralizes it. Compare with matched clean pairs.

**Primary metric.** erroneous supervision artifacts per root example; fraction of modules contaminated; correlation structure across labels.

**Kill if.** The pipeline rejects/repairs most bad pairs or the error rarely propagates beyond the root SQL target.

**Why this fits the new process.** The first result is an artifact audit using real annotation errors; no new model is required.

### 5. Correlated-Teacher Failure

**Failure boundary.** Generator and selector can appear modular/independent while being trained from synthetic targets ultimately anchored to the same erroneous gold pair/teacher assumptions.

**Hypothesis.** Intermediate supervision reduces **error diversity**: when the generator makes a gold-induced mistake, the selector is unusually likely to endorse it because both saw correlated supervision.

**Cheap test.** On 20–50 audited bad/ambiguous training roots, inspect generated candidates and selector targets. Compare selector preference agreement on gold-induced errors versus independently created semantic mutants.

**Kill if.** Selector errors are no more correlated with generator/gold errors than on clean controls.

### 6. Reasoning-Structure Plan Tax

**Boundary explicitly noted by OpenSQL.** CTE-heavy explicit reasoning can improve generation but produce more expensive SQL.

**Hypothesis.** Accuracy-oriented intermediate supervision shifts generated SQL toward structures whose **database execution cost** is materially worse than semantically equivalent direct baselines.

**Cheap test.** Sample 30–50 OpenSQL-correct BIRD queries. Compare optimizer cost/runtime/plan operators against gold or a semantically equivalent compact rewrite, controlling for result correctness.

**Kill if.** Runtime/plan cost differences are negligible or DB optimizers erase most structural overhead.

**Claim restriction.** This is a model-to-DBMS interface boundary, not a new query optimizer.

---

## Cluster 3 — Entity matching quality versus analytical truth

### 7. DownstreamImpact-GEM

**Anchor:** GLEAM.

**Failure boundary.** Pairwise precision/recall/F1 treats each entity-pair mistake equally, but downstream joins and aggregates weight entities by frequency, degree and measure value.

**Hypothesis.** Entity-matching F1 is a weak predictor of **downstream analytical error**; small differences in F1 can produce large differences in aggregate answers.

**Cheap test.** Use GLEAM's seven public datasets. Construct 20–50 simple, data-grounded downstream queries using existing numeric/category attributes: count by category, sum/average where available, top-k frequency, group-by after deduplication. Sweep matching thresholds or existing baselines and correlate pairwise F1 with query-answer error.

**Kill if.** F1 strongly predicts downstream query error across datasets and thresholds.

**Prior-art guardrail.** Entity resolution is known to affect downstream analytics. The publishable object would require a sharp failure law/benchmark showing that current LLM-GEM evaluation selects the wrong operating point for analytical workloads.

### 8. High-Impact Tail Matching

**Failure boundary.** Errors on high-value or high-degree entities can dominate query bias while being almost invisible in macro F1.

**Hypothesis.** Matching error should be decomposed by **downstream influence**, and LLM matchers have a systematic tail risk on influential entities.

**Cheap test.** Rank entities by downstream degree/value/frequency using only existing dataset attributes. Compare error rates in top 1/5/10% influence quantiles against the rest.

**Kill if.** Error is approximately uniform by influence or influential-entity errors do not move query answers.

### 9. Feedback-Skew Adaptivity Trap

**Anchor mechanism under test:** GLEAM's online Bayesian/adaptive connector.

**Failure boundary.** Online feedback may come from the current query workload, which is usually nonuniform across entity/domain segments.

**Hypothesis.** Adaptive thresholds improve the hot segment while silently degrading cold segments, creating **workload-conditioned matching bias**.

**Cheap test.** Using labeled public data, reveal feedback in a deliberately skewed but realistic order (e.g., majority category first), then evaluate segment-level F1 and downstream query error on untouched categories. Repeat with random feedback order.

**Kill if.** Adaptive performance is order/segment invariant.

**No method yet.** Only after the phenomenon appears consider stratified feedback or uncertainty controls.

---

## Cluster 4 — Semantic query engines: average quality may hide unstable decisions

### 10. Semantic Repeatability Gap

**Anchor:** SemBench.

**Failure boundary.** LLM semantic operators are stochastic. Mean F1/relative error plus standard deviation does not directly measure whether a user-visible answer changes identity across repeated runs.

**Hypothesis.** Some semantic queries have acceptable average quality but high **decision instability**: top-1/top-k membership, rank order, threshold classification or returned join partners flip across runs.

**Cheap test.** Use SemBench's existing repeated-experiment harness on 20–50 queries. For each query define a decision-stability metric appropriate to its operator: Jaccard of result set, rank correlation/top-k turnover, label flip rate, join-partner consistency.

**Kill if.** Same-query decision stability is uniformly high after controlling model temperature and seed.

**Paper shape.** Reliability benchmark/metric first; a stabilizing execution strategy only if needed.

### 11. Operator-Composition Cliff

**Failure boundary.** Semantic operators are often evaluated as complete query pipelines, but error may grow nonlinearly with the number/type of composed semantic operations.

**Hypothesis.** Semantic query quality exhibits a **composition cliff** for particular operator pairs (e.g., semantic join→rank, filter→join, map→classify) that cannot be predicted from individual operator quality.

**Cheap test.** Use SemBench's 55 public queries. Annotate operator sequence/depth, then compare observed end-to-end quality with an independence-style expectation from simpler/single-operator queries where available. Focus on natural existing queries; do not manufacture a large synthetic benchmark first.

**Kill if.** Quality degradation is approximately additive/predictable from operator count alone.

### 12. Average-F1 / Decision-Regret Mismatch

**Failure boundary.** A one-point F1 difference may be irrelevant for some analytic decisions and catastrophic for others.

**Hypothesis.** System ranking by SemBench's average quality metrics can disagree with ranking by **decision regret** on top-k/thresholded analytical outputs.

**Cheap test.** Re-score existing SemBench system outputs with task-specific decision losses on queries where a natural decision exists. Compare system rank correlation with original quality ranking.

**Kill if.** Rankings are nearly identical.

---

## Cluster 5 — Sample-based rewrite validation may certify the wrong rewrite

### 13. SampledRewriteUnsoundness

**Anchor:** ReSequel.

**Failure boundary.** Candidate rewrites are verified on sampled data, but SQL equivalence is a universal property over relevant database states.

**Hypothesis.** A measurable subset of sample-accepted LLM rewrites are **false equivalences** that diverge on the full dataset or targeted rare/boundary tuples.

**Cheap test.** ReSequel already stores rewrite candidates/results and ships eight workloads. Select 30–50 accepted rewrites, run original and rewritten SQL on the full available database, then targeted slices containing NULLs, extreme values, rare groups and boundary dates when applicable.

**Kill if.** Zero/negligible divergences occur across a sufficiently adversarial but data-grounded set.

**Important prior art.** Query equivalence checking is not new. The research claim is empirical: does sampling-based validation in modern LLM rewrite systems create a real reliability hole, and which rewrite classes trigger it?

### 14. Verification-Sample Blind Spots

**Failure boundary.** Not all sample disagreements are equally likely; random/downsampled data may systematically erase joins, duplicates, rare groups and correlation structures that distinguish rewrites.

**Hypothesis.** False acceptance concentrates in a small set of **semantic blind-spot classes**: DISTINCT/duplicate sensitivity, outer-join null extension, non-key joins, rare predicates, grouping boundaries, ordering/limit ties.

**Cheap test.** If any false equivalences are found in Idea 13, classify them by SQL operator/failure family and compare to the frequency of those operators among all accepted rewrites.

**Kill if.** No enriched class appears or failures are random one-offs.

### 15. Speedup Fragility under Data Drift

**Boundary.** ReSequel's template-specific rules are learned using current catalog/statistical metadata.

**Hypothesis.** A rewrite can remain semantically correct yet lose its speed advantage—or become pathological—after ordinary data-distribution/scale changes, meaning `rewrite quality` is snapshot-dependent.

**Cheap test.** Use TPC-H/DSB scale factors and available benchmark variants. Reapply the same accepted rewrite at multiple sizes/distributions and compare speedup sign/magnitude.

**Kill if.** Rewrite speedups remain directionally stable across ordinary scale/distribution changes.

**Claim restriction.** Do not call this an LLM drift paper unless the effect is specifically stronger for LLM-generated rewrites than native optimizer rules.

---

## Cluster 6 — DBMS-guided repair has an observability boundary

### 16. Silent-Semantic Boundary of SafeQL

**Anchor:** SafeQL.

**Failure boundary.** DBMS feedback naturally exposes invalid syntax/schema/function/value references, but executable semantic errors generate no failure signal.

**Hypothesis.** SafeQL's gain is concentrated on **observable execution errors**, while executable-wrong cases remain largely unchanged; its apparent trustworthiness therefore depends strongly on error type distribution.

**Cheap test.** Collect 30–50 public BIRD/Spider incorrect model outputs, split into non-executable versus executable-wrong using gold/test-suite checks, and run the SafeQL repair path or reproduce its decision logic. Compare correction rate by class.

**Kill if.** SafeQL corrects executable semantic errors at a similar rate despite no DBMS error signal.

**Prior-art guardrail.** Do not propose a generic semantic verifier afterward unless a narrower failure mechanism emerges.

### 17. Repair-Locality Cliff

**Failure boundary.** Safe query-space search assumes the correct query is reachable through constrained component edits.

**Hypothesis.** Repair success falls sharply when the semantic correction requires a **non-local structural change** such as join-path replacement, aggregation-grain change, subquery introduction or correlated-condition rewrite.

**Cheap test.** For 30–50 repairable failures, compute a simple edit taxonomy from wrong SQL to gold/equivalent correct SQL: token-local, clause-local, join-structural, aggregation-structural. Measure repair success by category.

**Kill if.** No locality cliff exists.

---

## Cluster 7 — Synthetic SQL diversity may not equal learning-value diversity

### 18. DiversityUtility-SQL

**Anchor:** SQL-Factory.

**Failure boundary.** SQL-Factory optimizes structural/schema diversity and shows aggregate downstream gains, but a diversity metric is only useful if it predicts **which generated examples reduce downstream errors**.

**Hypothesis.** Some highly diverse generated queries add little learning value, while low-frequency examples aligned with real model failure modes contribute disproportionately.

**Cheap pre-training test.** Before any new training, map SQL-Factory's generated corpus and a held-out Text2SQL model's real errors into the same coarse structural/failure taxonomy (join hops, nestedness, aggregation, set ops, temporal, duplicate sensitivity, etc.). Measure whether corpus diversity covers the empirical error distribution or mostly explores low-error regions.

**Kill if.** Existing diversity/coverage scores already correlate strongly with downstream error coverage.

### 19. Synthetic-Coverage Mirage

**Failure boundary.** A corpus can cover many SQL skeletons while still missing **semantic boundary cases** that distinguish correct from plausible-wrong SQL.

**Hypothesis.** SQL-Factory has high structural diversity but low coverage of near-neighbor semantic contrasts such as DISTINCT vs non-DISTINCT, inner vs left join, date boundary, top-k ties, denominator population and aggregation grain.

**Cheap test.** Sample 50–100 generated queries and measure whether each has a naturally generated or existing minimally different semantic counterpart. Compare to real benchmark error pairs.

**Kill if.** Contrastive semantic coverage is already high.

### 20. Generated-Data Transfer Concentration

**Failure boundary.** Aggregate downstream improvement may be concentrated in a few schema/query families.

**Hypothesis.** Synthetic SQL helps primarily where generated examples overlap the target benchmark's structural motifs, and can show little transfer to under-covered failure classes.

**Cheap test.** Use reported/per-example downstream predictions if available, or reproduce a small existing checkpoint evaluation. Stratify gains by motif coverage without generating new training data.

**Kill if.** Gains are broad and insensitive to coverage.

---

# 3. Additional compact probes

| # | Probe | Anchor | First observation |
|---:|---|---|---|
| 21 | **Gold-Column Recall vs Relationship Recall** | schema subsetting | which metric better predicts EX? |
| 22 | **Schema Reopen Rescue Rate** | schema subsetting | fraction of failures fixed by access to omitted schema |
| 23 | **Subset Confidence Miscalibration** | schema subsetting | high-confidence subset misses required bridge? |
| 24 | **Bad-Gold Artifact Multiplier** | OpenSQL | number of wrong intermediate artifacts per audited bad pair |
| 25 | **Module Error Correlation Matrix** | OpenSQL | generator/linker/selector failures share root label? |
| 26 | **CTE Runtime Inflation Tail** | OpenSQL | p95/p99 plan/runtime ratio on correct queries |
| 27 | **F1-to-Aggregate Rank Inversion** | GLEAM | matcher A has higher F1 but worse query answers than B |
| 28 | **Influential Entity Error Rate** | GLEAM | top-value/degree entities versus rest |
| 29 | **Feedback-Order Sensitivity** | GLEAM | shuffled versus skewed online feedback |
| 30 | **Top-k Turnover Rate** | SemBench | repeated semantic rank/filter results |
| 31 | **Semantic Join Partner Stability** | SemBench | same entity joins to different partners across runs |
| 32 | **Composition Residual** | SemBench | observed pipeline error minus predicted component error |
| 33 | **Full-Data Rewrite Divergence** | ReSequel | sample-passed rewrites that fail full data |
| 34 | **Rare-Group Rewrite Stress** | ReSequel | divergence concentrated on rare groups/NULLs |
| 35 | **Rewrite Speedup Sign Flip** | ReSequel | speedup becomes slowdown across scale factors |
| 36 | **Executable-Wrong Repair Rate** | SafeQL | semantic failures with no DBMS error |
| 37 | **Structural Edit Radius vs Repair** | SafeQL | repair probability by edit class |
| 38 | **Synthetic Corpus Error-Coverage** | SQL-Factory | generated motif distribution versus real model errors |
| 39 | **Semantic Contrast Coverage** | SQL-Factory | availability of minimally different meaning pairs |
| 40 | **Downstream Gain Concentration** | SQL-Factory | where synthetic training actually helps |

---

# 4. Provisional ranking from this database-venue batch

This ranking is **not** a promise of novelty. It orders ideas by how cheaply they can expose a meaningful failure using existing artifacts.

| Rank | Idea | Why it is strong | Prior-art pressure | First empirical cost |
|---:|---|---|---|---|
| 1 | **SampledRewriteUnsoundness** | direct, falsifiable reliability boundary in a strong VLDB-2026 system; public full workloads | high on equivalence theory, low on this concrete failure audit | very low |
| 2 | **SupervisionFanout-SQL** | real annotation errors can be traced through a public 4-stage supervision pipeline without training | medium | very low |
| 3 | **Semantic Repeatability Gap** | public benchmark already has repeat harness; asks user-visible stability rather than mean F1 | medium | low |
| 4 | **DownstreamImpact-GEM** | challenges whether the accepted entity-matching metric selects the right system for analytics | medium-high from task-aware ER literature | low-medium |
| 5 | **JoinBridge Blind Spot** | explains why pruning fails using relational structure instead of another retriever | medium | blocked until artifact release |
| 6 | **Silent-Semantic Boundary of SafeQL** | cleanly separates observable DBMS errors from silent semantic errors | high | low |
| 7 | **Schema-Ambiguity Phase Transition** | attacks the causal interpretation of `large schema` with a measurable natural-data variable | medium | low once data available |
| 8 | **Operator-Composition Cliff** | system-level semantic operator reliability, grounded in public SemBench | medium | low |
| 9 | **Feedback-Skew Adaptivity Trap** | tests a concrete online-adaptation assumption in GLEAM | medium | low-medium |
| 10 | **DiversityUtility-SQL** | asks whether synthetic-diversity metrics predict actual learning need | high | low for diagnostic, high for causal training test |

---

# 5. Cross-anchor ideas that emerge only after comparing the papers

These are not methods yet; they are hypotheses suggested by repeated patterns across anchors.

## X1. Proxy Metric Failure as the research object

Across the anchors, each system relies on a proxy:

- schema subsetting → schema recall / context reduction;
- OpenSQL → correctness of generated intermediate supervision;
- GLEAM → pairwise F1;
- SemBench → average result quality/cost;
- ReSequel → sample agreement;
- SQL-Factory → corpus diversity/schema coverage.

A potentially broad research program is:

> **When does the proxy used to optimize an AI-data system cease to predict the downstream property users actually care about?**

The point is not to build a generic `proxy verifier`. Each anchor must first show its own proxy failure experimentally.

## X2. Information removal versus information corruption

Three anchors expose a recurring asymmetry:

- schema subsetting removes context;
- OpenSQL can multiply corrupted supervision;
- sample-based rewrite verification can incorrectly certify a transformation.

This suggests measuring **information damage type** rather than raw model error: missing evidence, misleading evidence, and falsely certified evidence may have very different downstream recovery properties.

Do not turn this into a universal framework before at least two anchor failures reproduce.

## X3. Component quality versus decision quality

GLEAM and SemBench both expose a gap between component-level metrics and end-user decisions. A matcher/filter/join can have good average quality while flipping a ranking, thresholded alert or aggregate conclusion.

A boundary paper could become viable if the same rank inversion appears independently in both entity matching and semantic-query engines.

---

# 6. Recommended next order

Still **do not design new architectures**. The cheapest evidence sequence is:

1. **ReSequel** — audit 30–50 sample-accepted rewrites on full data and rare/boundary slices.
2. **OpenSQL** — pass 20–50 known bad/ambiguous BIRD roots through augmentation and measure label fanout.
3. **SemBench** — rerun 20–50 queries several times and measure decision stability instead of only mean quality.
4. **GLEAM** — compare matcher F1 against downstream aggregate/rank error across thresholds/baselines.
5. **SafeQL** — stratify repair success by executable-wrong versus DBMS-error-visible failures.
6. **Schema subsetting** — begin only when the BigBird/SKALPEL artifact is publicly reproducible; otherwise keep BLOCKED.
7. **SQL-Factory** — first do corpus/error-distribution analysis; training comes only if the diagnostic shows a coverage gap.

**Hard stop rule:** if an anchor's proxy reliably predicts the downstream property, kill the derived idea. If the failure requires fabricated edge cases and does not appear on the public workload, do not promote it to a main paper claim.

---

# 7. Sources / reproducibility anchors

- VLDB 2026 program — Text-to-SQL, semantic-query and LLM/data sessions: https://vldb.org/2026/program.html
- Large-schema schema subsetting / BigBird / SKALPEL: DOI `10.14778/3819518.3819531`; authors' project page: https://adalabucsd.github.io/publications.html
- OpenSQL paper: https://www.vldb.org/pvldb/vol19/p1628-li.pdf
- OpenSQL code: https://github.com/TsinghuaDatabaseGroup/OpenSQL
- GLEAM / Generalized Entity Matching with Adaptivity via LLMs: DOI `10.1145/3802066`
- GLEAM code/data: https://github.com/Blondig/GLEAM
- SemBench paper: https://arxiv.org/abs/2511.01716
- SemBench code/data: https://github.com/SemBench/SemBench
- ReSequel: https://arxiv.org/abs/2606.20853
- ReSequel code/results: https://github.com/CoDS-GCS/ReSequel
- SafeQL: DOI `10.14778/3819518.3819545`; arXiv `2608.09260`
- SQL-Factory paper: https://www.vldb.org/pvldb/vol19/p292-gao.pdf
- SQL-Factory code: https://github.com/LJHzju/SQL-Factory
