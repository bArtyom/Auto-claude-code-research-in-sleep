# Non-SQL Framings: Research Directions That Stop Treating SQL as the Main Object

> These ideas intentionally reject “better SQL generation” as the primary research goal. The question is whether an agent can solve data reasoning tasks more reliably by changing the *interface, target object, learning unit, or evaluation unit*.

## 1. Text2Experiment, not Text2SQL

User intent is compiled into an experiment plan:

- hypothesis
- required observations
- controls
- confound checks
- derived quantities
- stopping rule

SQL is generated only as one instrument for executing the experiment.

### Example
Question: “Did the promotion improve retention?”

Instead of one SQL query, produce:

1. define treated cohort
2. define comparison cohort
3. check pre-period imbalance
4. estimate retention curves
5. sensitivity analysis
6. report uncertainty

### Thesis
Many “Text2SQL” failures are actually failures to recognize that the user asked for an analysis, not a database lookup.

---

## 2. Text2Dataflow

Generate a typed dataflow DAG instead of SQL:

`source -> filter -> join -> deduplicate -> aggregate -> compare -> output`

Then lower the DAG into SQL, Spark, pandas, dbt, or a warehouse-specific plan.

### Research advantage
The target is engine-independent and exposes intermediate semantics directly.

---

## 3. Text2Invariant

The main model does not generate SQL. It generates invariants describing what any valid answer must satisfy.

Examples:

- one row per customer
- revenue nonnegative after exclusions
- output customer count <= eligible customer count
- cohort partition sums to total

A separate synthesizer searches for SQL satisfying the invariant set.

### Extreme hypothesis
For difficult enterprise tasks, generating the *specification* may be easier than generating the program.

---

## 4. Text2SemanticContract

Output a machine-checkable contract:

```text
entity: customer
measure: recognized_revenue
currency: USD
grain: customer
window: previous_calendar_year
exclusions: refunded_orders
ranking: descending
limit: 10
```

The contract is the benchmark target; SQL is merely a backend implementation.

### New benchmark possibility
Score semantic contracts independently from SQL execution. This separates language understanding from physical implementation.

---

## 5. Text2QuestionDecomposition

The system is judged on whether it discovers the *minimal set of subquestions* required to answer the user.

Example:

“How many high-value customers churned?”

requires:

- what counts as high-value?
- what counts as churn?
- which time window?
- entity grain?

### Research object
The quality of decomposition and ambiguity discovery before any SQL exists.

---

## 6. Text2ClarificationPolicy

Do not optimize answer accuracy. Optimize the sequence of clarification questions that transforms an underspecified request into a fully specified analytical contract.

### Metric
Number of user bits required to reach a correct contract.

This becomes an interactive compression problem.

---

## 7. Text2EvidenceGraph

Output a graph whose nodes are semantic claims and whose edges point to supporting evidence:

- schema
- docs
- rows
- prior accepted reports
- user confirmations

SQL is accepted only if derivable from the graph.

### Benchmark
Can a human auditor reconstruct why each query clause exists?

---

## 8. Text2RelationalProof

Target a proof in relational algebra or logic that a proposed query implements the requested intent under explicit assumptions.

### Strong version
The model can fail with “unprovable under current metadata.”

This turns abstention into a formal outcome rather than a confidence heuristic.

---

## 9. Text2Simulation

For ambiguous analytics questions, build a tiny synthetic world where competing interpretations produce different outcomes. Ask the user which behavior matches intent.

### Example
Two candidate definitions of “active customer” are illustrated on a 5-row toy dataset.

### HCI angle
Users may answer behavioral examples more reliably than abstract clarification questions.

---

## 10. Text2Counterfactual

Instead of returning current database values, return a model of how the answer should change under controlled changes.

Questions like “what drives churn?” are better expressed as counterfactual structures than direct SQL.

### Research line
Detect when a user request is fundamentally causal and refuse to silently substitute correlational SQL.

---

## 11. Text2SemanticDiff

Given an existing trusted report/query and a new user request, generate the smallest semantic diff:

- time window changed
- metric changed
- cohort changed
- grouping added

Then apply that diff to the trusted artifact.

### Why useful
Many real analytics requests are modifications of known analyses, not greenfield programs.

### Evaluation
Diff correctness may be easier to verify than whole-query correctness.

---

## 12. Text2DataProduct

The agent creates a reusable governed object:

- dbt model
- metric definition
- tests
- documentation
- ownership metadata

rather than a transient SQL answer.

### Research question
Can an agent convert repeated ad-hoc questions into durable semantic assets and reduce future reasoning cost?

---

## 13. Text2QuerySuite

Return not one query but a suite:

1. main answer query
2. validation query
3. edge-case query
4. data-quality query
5. cost check

### Difference from verifier architecture
The query suite itself is the output specification. Reliability emerges from structured redundancy.

---

## 14. Text2ExecutableNotebook

Compile user intent into an auditable analytical notebook with named cells:

- assumptions
- exploration
- transformations
- validation
- result

### Benchmark shift
Evaluate trajectory transparency and reproducibility, not just final execution match.

---

## 15. Text2SemanticPatch

When a query is wrong, do not regenerate. Output a semantic patch:

```diff
- grain: order
+ grain: customer
- count(order_id)
+ count_distinct(customer_id)
```

Then deterministically compile the patch.

### Research angle
Study edit locality and whether agents can repair meaning without causing collateral changes.

---

## 16. Text2BoundaryCases

The primary task is to identify rows or hypothetical cases where the user’s intent is ambiguous.

### Example
Which of these customers should count as “churned”?

The agent surfaces 5 maximally informative boundary cases. User labels them, and a semantic rule is inferred.

### Connection
This reframes clarification as active concept learning.

---

## 17. Text2Policy

For recurring business concepts, infer a policy that maps records to semantic labels:

`customer -> active/inactive`

`order -> recognized/unrecognized revenue`

SQL is synthesized from the policy.

### Why interesting
Many business metrics are classification policies disguised as SQL expressions.

---

## 18. Text2OntologyEdit

Sometimes the real problem is missing semantic infrastructure. Let the agent propose edits to the organization’s ontology / metric layer rather than repeatedly solving the same ambiguity ad hoc.

### Long-term objective
Reduce future Text2SQL difficulty by changing the environment.

---

## 19. Text2WarehouseQuestion

Reverse the direction: given a user question and current metadata, generate the *best question to ask the warehouse owners* to make future tasks easier.

### Example
“Is `customer_status='A'` authoritative, or should active status be derived from last transaction?”

### Meta-learning angle
Agents can improve the information architecture of the warehouse itself.

---

## 20. Text2Decision, not Text2Answer

If the downstream goal is a business decision, SQL is intermediate. Model the full chain:

`question -> evidence -> decision options -> utility -> recommendation`

### Safety property
The agent must state when the database cannot identify the decision-relevant quantity.

---

# Research programs enabled by these reframings

## Program A — Specification-first analytics

Combine Text2SemanticContract + Text2Invariant + deterministic compilation.

Hypothesis: language models are better at producing auditable semantic specifications than end-to-end SQL.

## Program B — Interactive concept learning

Combine Text2BoundaryCases + Text2ClarificationPolicy + Text2Policy.

Hypothesis: users can teach latent business definitions with a few strategically selected examples.

## Program C — Analytics asset compiler

Combine Text2SemanticDiff + Text2DataProduct + organizational memory.

Hypothesis: long-lived agents should reduce future query entropy by converting repeated analysis into governed reusable assets.

## Program D — Epistemic analytics

Combine Text2EvidenceGraph + Text2RelationalProof + explicit unprovability.

Hypothesis: the key reliability problem is not SQL synthesis but knowing which conclusions are actually supported by available organizational evidence.

## Program E — Environment-improving agent

Combine Text2OntologyEdit + Text2WarehouseQuestion + repeated-use learning.

Hypothesis: the best long-term database agent changes the warehouse’s semantic interface so that future tasks become easier and less ambiguous.

# Most radical thesis

A successful “Text2SQL agent” may eventually generate **less SQL over time**.

As it learns the environment, it should create semantic contracts, reusable metrics, validated data products, governance metadata, and clarifications that eliminate recurring ambiguity. Success is therefore not only higher accuracy per query; it is a decreasing *future semantic workload* for the organization.