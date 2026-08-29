# Idea Pool Expansion — Text2SQL × Data Agents

**Date:** 2026-08-26  
**Branch:** `research/text2sql-agent-fresh-2026-08-24`  
**Role:** idea-pool expansion after the first fresh literature/novelty pass  
**Empirical pilots:** NOT RUN  
**Formal cross-model reviewer:** UNAVAILABLE in this connector session; novelty scores are provisional.

## 0. Goal of this expansion

The first fresh pass already removed many crowded directions: generic database probing, multi-turn RL, planner/critic/verifier loops, ordinary semantic memory, semantic IR/compilers, generic clarification, candidate voting, ordinary repair, generic RBAC enforcement, generic no-answer refusal, and generic AI-SQL optimization.

This expansion therefore asks a stricter question:

> What *new scientific variable* appears when an LLM/agent becomes a persistent analytical interface to a real database, rather than merely a SQL generator?

The literature added in this round suggests several under-explored boundaries:

1. **Correct SQL does not imply a complete answer.** Classical database theory distinguishes closed/open/partially closed worlds and studies query completeness, certain answers, and complete approximations. Text2SQL systems largely evaluate correctness against a fixed database as if absence implies falsehood.
2. **Policy-compliant SQL does not imply an epistemically valid claim.** A July-2026 RBAC Text2SQL benchmark evaluates utility and policy compliance, but row-level security can silently hide rows; a globally phrased answer may therefore be wrong even when the SQL is perfectly authorized.
3. **Adaptive exploration does not imply statistically valid discovery.** Data agents can issue many sequential SQL probes and report the most interesting pattern. Classical adaptive-data-analysis theory and 2026 work on LLM p-hacking show why this can inflate false discoveries.
4. **A correct answer can still be fragile.** Missing data, delayed ingestion, changing snapshots, or a small number of influential tuples may flip a rank/decision without changing SQL correctness.
5. **Data-quality metadata is not yet answer semantics.** Modern platforms expose freshness, completeness, uniqueness, lineage and tests, but data agents usually consume them as context rather than propagate failed quality guarantees into what claims are allowed.
6. **Agent exploration has database externalities.** Token/LLM budgets are increasingly optimized, yet repeated probes can scan hot partitions, perturb caches, consume concurrency slots, and harm production workloads.
7. **One canonical business definition is not always correct.** Real organizations legitimately maintain different metric semantics by team, jurisdiction, product, policy epoch, or decision context.

These observations produce a second pool of ideas that are intentionally different from ordinary Text2SQL architecture work.

---

# 1. New literature collisions and guardrails

## 1.1 Incomplete databases are old theory, not new novelty

Foundational work on incomplete databases, partial closed-world assumptions, query completeness, certain/best answers, and complete approximations already exists. Recent KR 2026 work even studies almost-certain query answering over incomplete relational/graph data.

**Therefore:** do not claim `certain answers`, `query completeness`, or `open-world databases` as new. The potentially new contribution is making these notions operational in **natural-language database agents**, with metadata acquisition, scope-aware answer generation, and realistic evaluation.

## 1.2 Generic policy-aware Text2SQL is now crowded

`Benchmarking Text-to-SQL under Role-Based Access Control` (arXiv:2607.22115) directly evaluates Text2SQL under realistic RBAC constraints and separates SQL utility from policy compliance. SecureMCP, enterprise governed-analytics systems, and production RLS architectures also cover enforcement.

**Therefore:** the remaining gap is not `make SQL obey RBAC`. It is whether the **authorized visible world is sufficient for the semantic claim the user asked for**.

## 1.3 Generic transactional agents are newly crowded

`Agentic Transaction: Towards ACID-Compliant Agent Systems` (arXiv:2608.13900) and `Cordon: Semantic Transactions for Tool-Using LLM Agents` (arXiv:2606.17573) already import transactional semantics into agent runtimes.

**Therefore:** a generic `ACID for SQL agents` idea should be killed. A narrower open question is **read-only analytical coherence across multiple snapshots/sources**, especially heterogeneous workspaces that cannot share one DB transaction.

## 1.4 Generic “agents p-hack” is also no longer enough

2026 work directly studies p-hacking/sycophancy in coding agents and proposes mitigation protocols; classical adaptive-data-analysis theory already provides reusable holdouts and validity guarantees.

**Therefore:** the new object must be the **SQL exploration episode itself**: each adaptively chosen DB query changes the statistical validity of later claims.

## 1.5 Generic data-agent security is crowded

`Data Agents Under Attack` (arXiv:2606.08661), SEAgent, ToolPrivBench, SecureMCP, BridgeScope and related systems cover privilege misuse, injection, policy enforcement, and multi-step security.

**Therefore:** security ideas below must target a distinct semantic failure, such as **RLS-induced false global claims** or **cross-role memory contamination**, not generic injection/privilege escalation.

---

# 2. New Raw Idea Bank — 50 Additional Candidates

## A. Epistemic completeness / open-world semantics

| # | Idea | Core variable | Initial status |
|---:|---|---|---|
| A1 | **CompletenessProof-SQL** | machine-checkable query-answer completeness certificate | HIGH |
| A2 | **Zero-or-Unknown SQL** | distinguish true zero from missing/unobservable data | HIGH as benchmark/component |
| A3 | **MaximalSafeAnswer-SQL** | return the largest scope whose answer is guaranteed complete | HIGH |
| A4 | Certain/Possible Answer Agent | answer sets or intervals under incomplete evidence | MEDIUM-HIGH |
| A5 | LowerBound-SQL | exploit monotone queries to return certified lower bounds | MEDIUM |
| A6 | **MonotonicityProof-SQL** | query monotonicity as an epistemic safety property | MEDIUM-HIGH |
| A7 | Coverage-Aware Clarification | ask for missing coverage/definition rather than generic clarification | MEDIUM |
| A8 | Completeness-Aware Source Planner | acquire the cheapest source that closes the answer-completeness gap | HIGH |
| A9 | Data-Absence Semantics Benchmark | closed-world vs open-world failure benchmark | HIGH benchmark candidate |
| A10 | Missingness-Propagation Algebra | propagate incompleteness through joins/aggregates | MEDIUM |

## B. Access-control epistemics

| # | Idea | Core variable | Initial status |
|---:|---|---|---|
| B1 | **ViewpointSafe-SQL** | whether authorized visible data is sufficient for the requested claim | HIGH |
| B2 | RLS-World-Fallacy Benchmark | policy-compliant but globally false answers | HIGH benchmark candidate |
| B3 | ScopedWorld Rewriter | rewrite global question into the maximal authorized complete scope | HIGH |
| B4 | **Privilege-Scoped Memory** | memory learned under one privilege context contaminates another | MEDIUM-HIGH |
| B5 | Role-Change Memory Audit | behavior when user role changes mid-history | MEDIUM |
| B6 | Cumulative Aggregate Leakage Budget | sequence-level inference leakage from individually legal queries | MEDIUM; security collision risk |
| B7 | Policy-Conditioned Evidence Ledger | proof that each final claim is derivable from accessible evidence | MEDIUM; provenance collision risk |
| B8 | Authorization Counterfactual | identify the minimum extra permission needed to answer safely | MEDIUM-HIGH |

## C. Statistical validity of autonomous analytics

| # | Idea | Core variable | Initial status |
|---:|---|---|---|
| C1 | **AdaptiveInference-SQL** | valid statistical inference after adaptively chosen SQL probes | VERY HIGH |
| C2 | **DataDredgeBench** | false-discovery rate of open-ended analytics agents on null data | VERY HIGH benchmark candidate |
| C3 | ReusableHoldout-DBAgent | hidden confirmation slice for autonomous DB exploration | HIGH |
| C4 | AnytimeSQL | e-values/anytime-valid confidence for sequential analytics | HIGH |
| C5 | SpecificationLedger-SQL | log all attempted specifications/subgroups and adjust inference | HIGH |
| C6 | Exploration→Confirmation Compiler | separate exploratory SQL from confirmatory SQL automatically | HIGH |
| C7 | FDR-Aware Insight Agent | control false discovery rate across autonomous insight mining | HIGH |
| C8 | SubgroupStability-SQL | require holdout confirmation for discovered subgroup effects | MEDIUM-HIGH |
| C9 | SimpsonGuard-Agent | flag aggregate conclusions that reverse under justified stratification | MEDIUM-HIGH |
| C10 | Statistical Degrees-of-Freedom Meter | quantify how much analytical search occurred before a claim | HIGH metric candidate |

## D. Robustness of answers, not SQL

| # | Idea | Core variable | Initial status |
|---:|---|---|---|
| D1 | **DecisionMargin-SQL** | whether a user decision is stable across plausible missing data | HIGH |
| D2 | **RankFragility-SQL** | minimum plausible data perturbation that flips top-k/ranking | HIGH |
| D3 | Counterfactual Answer Margin | smallest tuple/value change that flips final answer | MEDIUM-HIGH |
| D4 | RobustInterval-SQL | certified value interval from missingness/freshness assumptions | HIGH |
| D5 | Answer Stability Certificate | bootstrap/subsample/source-drop stability of analytical claim | MEDIUM; PCS collision risk |
| D6 | Sensitivity-Directed Probe Agent | probe only data dimensions capable of flipping the answer | HIGH |
| D7 | Near-Boundary Clarification | clarify only when plausible worlds straddle a decision threshold | HIGH |
| D8 | Stable-Decision / Unstable-Number Interface | answer the decision when numeric estimate is uncertain but conclusion invariant | HIGH |

## E. Temporal and multi-source coherence

| # | Idea | Core variable | Initial status |
|---:|---|---|---|
| E1 | **TemporalCoherence-Agent** | final synthesis must refer to a coherent as-of world | MEDIUM-HIGH; transaction collision |
| E2 | Cross-Source Snapshot Benchmark | staggered updates across DB/files/docs create impossible synthesized worlds | HIGH benchmark candidate |
| E3 | FreshnessProof-SQL | claim validity under ETL/source freshness windows | HIGH |
| E4 | Event-Time / Processing-Time Guard | distinguish business event time from ingestion/processing time | HIGH |
| E5 | Read-Set Revalidation | recheck evidence dependencies before finalizing long-horizon answer | MEDIUM |
| E6 | ReplicaAware-SQL | choose primary/replica/sample using required freshness semantics | MEDIUM-HIGH |
| E7 | Temporal Conflict Resolver | detect two sources that are individually correct but refer to different business epochs | HIGH |

## F. Data-quality semantics and organizational meaning

| # | Idea | Core variable | Initial status |
|---:|---|---|---|
| F1 | **QualityTaint-SQL** | propagate failed freshness/uniqueness/completeness tests into allowed claims | HIGH |
| F2 | Quality-Aware Source Substitution | switch to alternate lineage path when authoritative source fails quality tests | MEDIUM-HIGH |
| F3 | Claim-to-Test Dependency Graph | identify exactly which dbt/data-quality assertions support an answer | HIGH |
| F4 | **SemanticJurisdiction-SQL** | same metric name can have multiple legitimate definitions scoped by team/task/time | HIGH |
| F5 | Definition Conflict Graph | preserve conflicting metric definitions instead of collapsing to one embedding | HIGH |
| F6 | Authority-Aware Semantic Retrieval | rank definitions by ownership/jurisdiction, not only semantic similarity | HIGH |
| F7 | Cross-Team Memory Contamination | finance/sales/product semantic memories cause negative transfer across contexts | HIGH |
| F8 | MeasurementType-SQL | infer nominal/ordinal/interval/ratio/unit semantics and block invalid aggregation | MEDIUM-HIGH |
| F9 | Currency-Time Type System | currency conversion must include rate source and valid time | MEDIUM |
| F10 | DenominatorFragility-SQL | expose whether metric conclusions depend on a hidden/unstable denominator definition | MEDIUM-HIGH |

## G. Operational externalities of agent exploration

| # | Idea | Core variable | Initial status |
|---:|---|---|---|
| G1 | **NonInvasive-SQL** | information gain per unit of production-workload interference | HIGH |
| G2 | ShadowProbe-Agent | explore on samples/read replicas, escalate only when fidelity is insufficient | HIGH |
| G3 | WorkloadExternality Benchmark | p95 latency / concurrency harm caused by autonomous probe sequences | HIGH benchmark candidate |
| G4 | QueryFootprint Contract | bound scans, partitions, locks, temp space and concurrency across an episode | MEDIUM-HIGH |
| G5 | Production-Aware Observation Planner | optimize DB-resource cost jointly with token/LLM cost | MEDIUM-HIGH |
| G6 | ObserverEffect-SQL | quantify how agent probes change caches/statistics/operational environment | WILD / SYSTEMS |
| G7 | Episode-Level Query Budget | per-query limits are insufficient; bound cumulative scan/resource use | MEDIUM-HIGH |

## H. Heterogeneous-source authority and partial failure

| # | Idea | Core variable | Initial status |
|---:|---|---|---|
| H1 | **SourceAuthority-Agent** | conflicting evidence should be resolved by authority/validity policy, not nearest retrieval | HIGH |
| H2 | FederatedPartialAnswer | maximal safe answer when one source is unavailable | HIGH |
| H3 | Source-Conflict Certificate | surface irreconcilable authoritative-source disagreements | HIGH |
| H4 | Heterogeneous Coverage Proof | prove which parts of a claim are covered by DB/PDF/API/file evidence | MEDIUM-HIGH |
| H5 | Modality Freshness Arbitration | stale PDF vs live DB vs semantic-layer definition | HIGH |
| H6 | Source Replacement Robustness | whether answer remains invariant when one source implementation changes | MEDIUM |

---

# 3. Twelve Strongest New Paper-Shaped Bets

## 3.1 CompletenessProof-SQL — query correctness is not answer completeness

### Problem essence

Text2SQL evaluation implicitly treats the database as the world. Real warehouses are often only a **partial observation** of the world because of delayed ingestion, partial integrations, retention limits, source outages, permissions, or intentional sampling.

A perfectly correct SQL query can therefore produce a confidently incomplete answer.

### Mechanism

Represent database/source metadata as partial-closed-world statements, freshness/coverage contracts, or learned-but-verifiable coverage assertions. For a generated SQL query, derive an answer status such as:

```text
COMPLETE
SCOPE_COMPLETE(scope=...)
LOWER_BOUND
POSSIBLY_INCOMPLETE
UNDETERMINED
```

Optionally produce a minimal missing-coverage certificate.

### Closest known work

Classical query-completeness/certain-answer literature; recent KR-2026 almost-certain query answering. No direct Text2SQL/data-agent method was found in this pass whose central object is a natural-language agent's **answer-completeness certificate**.

### Minimum falsification

Create complete databases, hide controlled subsets using realistic mechanisms, and ask identical NL questions. Compare normal Text2SQL confidence/data-quality prompting against completeness reasoning.

### Kill if

A trivial freshness/null-rate heuristic identifies the same incomplete-answer cases, or realistic enterprise metadata cannot support useful completeness statements.

**Provisional novelty:** 8.5/10.

---

## 3.2 ViewpointSafe-SQL — authorized data can still support the wrong worldview

### Problem essence

RLS can silently hide rows. Suppose a regional manager asks:

> Which region has the highest revenue?

Their SQL may be policy-compliant and return `East`, because only East is visible. Saying `East is the highest-revenue region` is nevertheless epistemically unjustified.

### Mechanism

Given `(question, role/policy, authorized view)`, determine whether the visible relation is **query-complete for the semantic scope of the question**. If not, either:

- rewrite the answer scope (`Among regions you are authorized to view...`),
- produce the maximal complete authorized query, or
- issue a permission/completeness certificate.

### Closest known work

The July-2026 RBAC Text2SQL benchmark focuses on policy compliance and whether queries can be answered with permitted data. The proposed delta is specifically **semantic claim sufficiency under silent filtering**, not authorization enforcement.

### Minimum falsification

Augment RBAC tasks with global-vs-local question pairs where the same authorized SQL result supports a scoped claim but not a global one.

### Kill if

Existing RBAC evaluation already captures this failure with equivalent semantics, or a fixed response prefix (`within your accessible data`) solves nearly all cases without reasoning.

**Provisional novelty:** 8.0/10.

---

## 3.3 AdaptiveInference-SQL — an autonomous data agent is an adaptive statistician

### Problem essence

An open-ended agent can issue dozens of SQL queries, inspect results, change hypotheses, choose subgroups, and then report the strongest pattern. Even if every SQL query is correct, the final statistical claim can be invalid because the data were reused adaptively.

### Mechanism

Treat the full DB interaction episode as an adaptive statistical process. Options include:

- protected confirmation partitions;
- reusable holdout mechanisms;
- anytime-valid e-values/confidence sequences;
- explicit specification ledgers and multiplicity correction;
- exploration/confirmation mode transitions.

The contribution should be a **data-agent interaction protocol**, not a new significance test.

### Closest known work

Classical adaptive data analysis; 2026 work on LLM p-hacking and mitigation. The open gap is SQL/data-agent episodes where the model itself chooses a long adaptive sequence of database queries.

### Minimum falsification

Generate null datasets with many plausible dimensions and ask agents to `find interesting/significant insights`. Measure episode-level false discovery rate and whether validity protocols reduce it while retaining power on planted-signal data.

### Kill if

Simple holdout splitting or logging all hypotheses solves the issue with no agent-specific mechanism.

**Provisional novelty:** 8.0–8.5/10.

---

## 3.4 MaximalSafeAnswer-SQL — do more than abstain

### Problem essence

When the requested answer is incomplete, refusing entirely wastes valid information. Database theory already studies complete approximations.

### Mechanism

Return the **largest useful sub-question/scope** whose answer can be certified complete, e.g.:

> Global churn is not answerable because APAC ingestion is missing. For US+EU through Aug 25, churn is 4.2%.

Search over scope restrictions, source subsets, time windows, or dimension values subject to completeness guarantees.

### Closest known work

Classical complete approximations over partially complete databases. Novelty must come from automatic NL intent preservation, enterprise metadata acquisition, and user-facing agent interaction.

### Minimum falsification

Paired tasks where the global query is incomplete but a valuable nearby scope is complete. Compare refusal, generic clarification, and maximal-safe reformulation.

### Kill if

The complete approximation is usually trivial/useless or cannot preserve user intent better than a generic clarification prompt.

**Provisional novelty:** 7.5–8.0/10.

---

## 3.5 DecisionMargin-SQL — sometimes the decision is certain even when the number is not

### Problem essence

Users often need a decision/ranking, not an exact scalar. Missing or uncertain data may make `revenue = 12.7M` uncertain while still making `Product A > Product B` guaranteed under every plausible completion.

### Mechanism

Construct a set of admissible database completions/perturbations from coverage or uncertainty assumptions. Compute whether the **decision functional** (winner, threshold crossing, ordering, allocation choice) is invariant.

Return:

```text
number: uncertain [12.1M, 13.4M]
decision: stable (A remains > B)
margin: minimum perturbation required to flip = ...
```

### Closest known work

Incomplete-database certain/best answers and query sensitivity exist, but this pass found no current Text2SQL/data-agent system centered on **decision invariance under data uncertainty**.

### Minimum falsification

Synthetic and semi-real business tasks with controlled missingness; compare exact-answer abstention to decision-level robust answering.

### Kill if

Decision stability is rare or bounds are so loose that almost all tasks collapse to `unknown`.

**Provisional novelty:** 7.5–8.0/10.

---

## 3.6 SemanticJurisdiction-SQL — business semantics are scoped, not globally canonical

### Problem essence

Semantic layers often assume one governed definition per metric. Real organizations may legitimately use multiple definitions:

- finance revenue vs sales booked revenue;
- active user for growth vs support;
- customer at account vs legal-entity level.

The issue is not ambiguity to eliminate; it is **jurisdictional polysemy**.

### Mechanism

Attach each semantic definition to scope metadata:

```text
metric
owner/team
valid-time
business process
allowed dimensions/grain
source authority
supersedes / conflicts-with
```

Resolve queries by jurisdiction and surface conflicts rather than collapsing all definitions into a single embedding.

### Closest known work

2026 semantic-layer systems and dbt materials explicitly acknowledge multiple possible business meanings, but current systems largely emphasize governed canonical definitions. No direct benchmark/method for **legitimate multi-definition semantic jurisdiction** was found in this pass.

### Minimum falsification

Construct multi-department workloads with deliberately conflicting-but-legitimate definitions. Compare global semantic layer, role prompt, nearest-definition RAG, and jurisdiction-aware retrieval.

### Kill if

Role/team metadata alone resolves almost every case, making the graph/jurisdiction machinery unnecessary.

**Provisional novelty:** 7.5/10.

---

## 3.7 QualityTaint-SQL — data quality failures should restrict claims

### Problem essence

Data agents can generate correct SQL against tables whose upstream freshness, uniqueness, or completeness tests are failing. Modern catalogs expose these tests, but the final narrative often still treats returned rows as fully trustworthy.

### Mechanism

Propagate data-quality assertions through query lineage. A failed test taints only claims that depend on the affected region/column/time range. The agent can:

- scope the claim;
- choose an alternative trusted source;
- report a lower confidence/quality state;
- identify the exact upstream test blocking a trustworthy answer.

### Closest known work

Data platforms expose tests/lineage/freshness, and 2026 work studies AI-generated data-quality tests. A July-2026 conceptual framework also argues for quality of agentic data use. The delta must be **claim-level propagation and counterfactual impact**, not merely giving tests to the LLM.

### Minimum falsification

Inject realistic dbt/data-quality failures upstream while keeping query syntax valid. Measure whether systems continue making unqualified claims.

### Kill if

Simply placing failing test messages in the prompt matches the structured taint-propagation method.

**Provisional novelty:** 7.0–7.5/10.

---

## 3.8 NonInvasive-SQL — the agent is part of the production workload

### Problem essence

Exploration policies optimize tokens and correctness but often ignore the database externality of the probes themselves. Ten `safe` read queries can still destroy p95 latency through scans, locks, cache pollution, or warehouse credit consumption.

### Mechanism

Plan information acquisition under a **production interference budget**. Candidate observations can run against:

- catalog statistics;
- samples;
- materialized summaries;
- read replicas;
- primary tables.

Escalate only when cheaper/less invasive evidence is insufficient.

### Closest known work

BridgeScope includes an `EXPLAIN` gate and agent/resource frameworks bound costs, but this pass found no Text2SQL method whose main objective is **information gain vs production-workload degradation across an analytical episode**.

### Minimum falsification

Run a background OLTP/warehouse workload while an agent solves SQL ambiguity tasks. Compare accuracy against p95 latency inflation, bytes scanned, concurrency/credit use.

### Kill if

A simple per-query cost threshold or read-replica-only policy attains the same frontier.

**Provisional novelty:** 7.0–7.5/10.

---

## 3.9 TemporalCoherence-Agent — do the observations describe one world?

### Problem essence

A long-running data agent may query multiple databases, replicas, files, dashboards and docs whose timestamps differ. The final synthesis can combine facts that were never simultaneously true.

### Mechanism

Every observation carries `(valid_time, observed_time, freshness SLA, source version)`. Before synthesis, solve a temporal-coherence constraint: is there an `as_of` world for which all claims are jointly valid? If not, repair by re-querying, narrowing time, or producing explicit temporal conflict.

### Closest known work

Agentic Transaction and Cordon make generic transactional semantics crowded. The narrower potential gap is **read-only, heterogeneous, non-transactional analytical evidence** across DataSpace/DAB-style workspaces.

### Minimum falsification

Controlled staggered updates across a database, CSV export, semantic definition, and document. Test whether agents synthesize impossible cross-time answers.

### Kill if

Basic timestamp normalization and `AS OF` prompting solves nearly all failures.

**Provisional novelty:** 6.5–7.0/10.

---

## 3.10 MeasurementType-SQL — SQL type correctness is weaker than measurement validity

### Problem essence

SQL permits operations that are mathematically executable but scientifically meaningless: averaging identifiers or ordinal categories, summing percentages with incompatible denominators, mixing currencies, comparing timestamps from different event semantics.

### Mechanism

Infer/maintain a lightweight measurement type system:

```text
Nominal / Ordinal / Interval / Ratio
Unit<USD>
Currency<USD, rate_date>
Percentage<denominator=eligible_users>
Time<EventTime>
Time<IngestionTime>
```

Type-check the semantic plan and require explicit coercions/conversions.

### Closest known work

Semantic layers encode metric definitions and units can exist in BI systems, but no recent Text2SQL paper found in this pass makes **measurement-theory validity** a first-class benchmark/mechanism.

### Minimum falsification

Create executable-but-invalid SQL mutants involving scale/unit/denominator/time-type mistakes. Compare plain semantic layers and LLM reviewers against measurement typing.

### Kill if

A curated semantic layer already prevents nearly all errors, making the proposed type system redundant.

**Provisional novelty:** 7.0/10.

---

## 3.11 Privilege-Scoped Memory — yesterday's authorized memory may be today's leak

### Problem essence

Text2SQL memory systems reuse prior semantic programs/traces. If a high-privilege session writes memory and a lower-privilege session retrieves it, the eventual SQL can still obey RLS while the **memory text/semantic knowledge itself** leaks restricted information or induces inaccessible assumptions.

### Mechanism

Associate every memory item with a security lattice / derivation scope. Retrieval must satisfy noninterference: the user may receive/use the memory only if its semantic content is derivable under current privileges. Support downgrading via safe abstraction where possible.

### Closest known work

General SEAgent/MAC work includes security-enhanced memory and information-flow graphs; ToolPrivBench studies privilege selection. The Text2SQL-specific delta would be **semantic-memory reuse across database roles**, so novelty risk is significant.

### Minimum falsification

Create histories under admin/analyst/tenant roles, then switch roles and measure direct leakage plus semantic negative transfer.

### Kill if

Simple role-tag filtering of memory eliminates virtually all failures.

**Provisional novelty:** 6.5–7.0/10.

---

## 3.12 SourceAuthority-Agent — retrieval relevance is not epistemic authority

### Problem essence

DataSpace-style agents may find multiple plausible sources: a live warehouse table, a stale CSV export, a dashboard note, a finance PDF, and a semantic-layer metric. Nearest/relevant retrieval does not say which source is authoritative for the requested claim.

### Mechanism

Maintain source authority and validity metadata:

```text
owner
system_of_record
valid_time/freshness
derivation lineage
metric/domain jurisdiction
supersedes/conflicts-with
```

Planning optimizes not only relevance but **authority adequacy for the claim type**.

### Closest known work

Heterogeneous data-agent benchmarks test discovery and joins; semantic/catalog systems expose ownership/lineage. The potential contribution is explicit **authority-aware evidence selection under conflict**.

### Minimum falsification

Construct workspaces with highly relevant stale/non-authoritative distractors and less lexically similar authoritative sources.

### Kill if

Simple source-priority metadata or recency ranking matches the full method.

**Provisional novelty:** 7.0–7.5/10.

---

# 4. Strong Composite Directions

These are not additional modules; they combine orthogonal research variables into larger paper programs.

## X1. Epistemic SQL Agent

```text
question
  -> semantic scope
  -> authorized world
  -> data completeness
  -> quality/freshness status
  -> query
  -> answer stability
  -> final claim scope
```

The agent is evaluated on **what it is justified to say**, not merely whether its SQL matches gold.

Best components: CompletenessProof + ViewpointSafe + QualityTaint + DecisionMargin.

## X2. Statistically Valid Autonomous Analyst

```text
exploration mode
  -> adaptive SQL probes
  -> specification ledger
  -> discovery candidates
  -> protected confirmation / anytime-valid inference
  -> claim certificate
```

Core metric: episode-level false discovery rate at matched discovery power and query budget.

## X3. Incomplete-World Text2SQL Benchmark

Counterfactually manipulate the same database/task along axes:

- missing tuples;
- stale partitions;
- RLS-hidden rows;
- unavailable source;
- delayed dimension table;
- known-vs-unknown completeness metadata.

Then evaluate whether systems distinguish `false`, `zero`, `unknown`, `incomplete`, and `scope-complete`.

## X4. Governed Multi-World Data Agent

Treat `(role, time, source authority, semantic jurisdiction)` as the **world context**. The same natural-language question legitimately has different answers/definitions across contexts; the agent must never silently mix them.

## X5. Non-Invasive Scientific Data Agent

Combine adaptive inference with production externality:

> discover statistically valid insights while minimizing both database interference and model/tool cost.

This is harder, but it connects DBMS workload management with statistical validity rather than adding another reasoning loop.

---

# 5. Provisional Re-ranking of New Ideas Only

| Rank | New idea | Novelty | Feasibility | Main risk |
|---:|---|---:|---:|---|
| 1 | **CompletenessProof-SQL** | 8.5 | 8.0 | practical completeness metadata may be sparse |
| 2 | **AdaptiveInference-SQL / DataDredgeBench** | 8.0–8.5 | 8.0 | general agent p-hacking work may rapidly move here |
| 3 | **ViewpointSafe-SQL** | 8.0 | 8.5 | RBAC benchmark may already cover more than abstract reveals |
| 4 | **MaximalSafeAnswer-SQL** | 7.5–8.0 | 7.5 | classical completeness approximation is close methodologically |
| 5 | **DecisionMargin-SQL** | 7.5–8.0 | 7.0 | robust/incomplete-query literature may contain closer precedents |
| 6 | **SemanticJurisdiction-SQL** | 7.5 | 8.0 | may collapse to scoped semantic-layer metadata |
| 7 | **QualityTaint-SQL** | 7.0–7.5 | 8.5 | product systems already expose tests/lineage |
| 8 | **NonInvasive-SQL** | 7.0–7.5 | 7.0 | per-query cost gates / replica routing may be sufficient |
| 9 | **SourceAuthority-Agent** | 7.0–7.5 | 8.0 | catalog ranking may solve much of it |
| 10 | **MeasurementType-SQL** | 7.0 | 8.0 | curated semantic layers may already encode enough |
| 11 | **TemporalCoherence-Agent** | 6.5–7.0 | 7.5 | Agentic Transaction/Cordon collision |
| 12 | **Privilege-Scoped Memory** | 6.5–7.0 | 8.0 | general information-flow/MAC memory work collision |

---

# 6. Recommended Deep-Novelty Order

Before any experiment, the best next literature searches are:

1. **CompletenessProof-SQL:** Text2SQL + incomplete DB + query completeness + certain/best answers + completeness statements.
2. **AdaptiveInference-SQL:** adaptive data analysis + autonomous analytics + LLM p-hacking + reusable holdout + anytime-valid inference.
3. **ViewpointSafe-SQL:** RBAC Text2SQL full paper + inference-control / authorized-view completeness literature.
4. **DecisionMargin-SQL:** query sensitivity, robust query processing, database causality/responsibility, incomplete-data decision bounds.
5. **SemanticJurisdiction-SQL:** semantic layers, metric stores, contextual ontologies, multi-stakeholder/organizational semantics.
6. **QualityTaint-SQL:** data contracts, dbt tests, quality propagation, provenance semirings, claim-level reliability.
7. **NonInvasive-SQL:** query workload management, cost-based exploration, read replicas/AQP, production-safe agents.
8. **MeasurementType-SQL:** units/dimensional analysis, measurement scales, typed relational algebra, semantic metrics.

Do **not** run pilots until these eight have undergone deeper novelty checks. The goal of this round is pool expansion, not selecting a winner prematurely.

---

# 7. Key references used in this expansion

Recent/agent-side anchors:

- `DataSpace: Benchmarking Data Agents for Verifiable Analytics over Heterogeneous Workspaces`, arXiv:2608.03451.
- `Agentic Transaction: Towards ACID-Compliant Agent Systems`, arXiv:2608.13900.
- `Cordon: Semantic Transactions for Tool-Using LLM Agents`, arXiv:2606.17573.
- `Benchmarking Text-to-SQL under Role-Based Access Control`, arXiv:2607.22115.
- `Data Agents Under Attack: Vulnerabilities in LLM-Driven Analytical Systems`, arXiv:2606.08661.
- `AgentSM: Semantic Memory for Agentic Text-to-SQL`, arXiv:2601.15709.
- `Is Agent Memory a Database?`, arXiv:2605.26252.
- `Mitigating LLM-based p-Hacking by Preregistering for the Next LLM`, arXiv:2606.27687.
- 2026 working paper `Do Claude Code and Codex P-Hack?`.
- `A Multi-Layer Testing Framework for Automated Data Quality Assurance in Cloud-Native ELT Pipelines`, arXiv:2605.20500.

Foundational/database anchors:

- Levy, `Obtaining Complete Answers from Incomplete Databases`, VLDB 1996.
- Nutt/Razniewski et al., query/table completeness and partial closed-world work.
- Dwork et al., `Preserving Statistical Validity in Adaptive Data Analysis`, 2014/2015.
- Gheerbrant et al., certain/best answers and tractable rewritings over incomplete data.
- `Almost Certain Query Answering over Incomplete Relational and Graph Data`, KR 2026.

The central discipline for the next round is unchanged: a classical idea transferred to agents is not sufficient novelty by itself. It survives only if the agent setting creates a new observable failure mode, a new decision variable, or a substantially different empirical object.