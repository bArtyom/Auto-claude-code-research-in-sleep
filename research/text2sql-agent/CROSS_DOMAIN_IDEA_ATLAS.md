# Cross-Domain Idea Atlas: Text2SQL × Agents

> Goal: deliberately import mature ideas from mathematics, program synthesis, theorem proving, statistics, NLP, vision, RL, causal inference, control, and software testing into Text2SQL. The point is not analogy for analogy's sake: each transfer below is written as a falsifiable mechanism.

## 0. Meta-thesis

Text2SQL has historically been framed as conditional generation:

`(question, schema) -> SQL`

A more useful abstraction is **sequential scientific inference over executable programs**:

`latent intent + latent schema semantics + latent data assumptions -> observations via tools -> posterior over programs -> proof/evidence -> decision`

Once phrased this way, a large body of classical methods becomes relevant. Three recurring patterns appear across domains:

1. **Search over hypotheses rather than trust one sample.**
2. **Acquire information specifically to eliminate uncertainty.**
3. **Separate proposal from verification and require a certificate before termination.**

That suggests a broader research agenda than “better SQL prompting”: build SQL agents that perform search, experiment design, verification, calibration, adaptation, and selective abstention.

---

# 1. Mathematics / Formal Methods / Program Synthesis

## Idea M1 — Proof-Carrying SQL

### Borrowed principle
In theorem proving and proof-carrying systems, a result is accepted only together with a checkable certificate.

### Text2SQL transfer
A SQL agent should output not only SQL, but a compact **semantic proof bundle**:

- intent decomposition
- selected entities/metrics/dimensions
- join-path justification
- aggregation-grain claim
- critical assumptions
- executable checks supporting each assumption
- unresolved obligations

The proof itself does not need to be formally complete. The key is that each important semantic choice maps to an independently checkable obligation.

Example:

```text
Claim: revenue is aggregated at customer-month grain
Obligation O1: one row per order in fact_orders
Obligation O2: customer join is many-to-one
Obligation O3: refunded orders are excluded by business definition
Obligation O4: metric uses recognized_revenue, not gross_amount
```

A lightweight verifier checks O1–O4 without reading the generator's hidden reasoning.

### Research question
Does certificate-gated finalization reduce silent semantic errors at fixed token/tool budget?

### Strong experiment
Take executable-but-wrong SQL pairs. Compare:

- generation only
- generation + generic critique
- generation + proof obligations
- generation + proof obligations + independent verifier

Measure silent-error rate, tool cost, and calibration.

---

## Idea M2 — CEGIS-SQL++: Counterexample-Guided Inductive Synthesis

### Borrowed principle
Counterexample-Guided Inductive Synthesis alternates between a learner proposing a program and a verifier producing a counterexample that invalidates it.

### Text2SQL transfer
Represent ambiguity as a version space of candidate SQL programs. Instead of asking “is this query correct?”, ask:

> What observation would eliminate the largest number of currently plausible programs?

The verifier designs a database probe or identifies rows on which candidate SQLs disagree.

### New twist beyond the existing CEGIS idea
Use **counterexample value** as an explicit objective:

`score(probe) = expected hypothesis elimination / probe cost`

This turns verification into information-efficient experimental design rather than ad hoc testing.

### Key metric
Hypotheses eliminated per tool call / per rows scanned.

---

## Idea M3 — Branch-and-Bound Semantic Search

### Borrowed principle
Branch-and-bound solves combinatorial optimization by expanding partial solutions only when their upper bound can beat the incumbent.

### Text2SQL transfer
Search over partial SQL ASTs or semantic plans:

```text
metric -> source relation -> join path -> filters -> grouping -> temporal semantics
```

Each partial plan gets:

- feasibility score
- evidence score
- semantic-risk lower bound
- execution-cost estimate

Prune branches that already violate schema, cardinality, business semantics, or cost constraints.

### Why interesting
LLM tree search is usually expensive because all branches are treated similarly. SQL has unusually rich deterministic structure, so aggressive symbolic pruning may make search practical.

### Falsifier
If deterministic bounds fail to prune a large fraction of wrong plans before SQL execution, the method is not worth the complexity.

---

## Idea M4 — A* Search over Semantic Plans

### Borrowed principle
A* combines cost-so-far with a heuristic estimate of remaining cost.

### Text2SQL transfer
Define state as a partially resolved semantic plan. Actions resolve one ambiguity: table, metric, join, filter, grain, time field, etc.

`f(state) = evidence/tool cost so far + estimated remaining semantic risk`

A learned heuristic predicts which unresolved choices are likely to create expensive downstream repair.

### Possible paper angle
**Risk-aware planning for Text2SQL**, where planning focuses first on high-impact ambiguities instead of following a fixed slot order.

---

## Idea M5 — Minimum Description Length SQL Selection

### Borrowed principle
MDL/Occam-style inference prefers the shortest explanation that adequately explains the observations.

### Text2SQL transfer
Among multiple execution-consistent SQL programs, prefer candidates minimizing:

`description length(SQL) + description length(unexplained assumptions) + penalty(test failures)`

The novelty is not simply “shorter SQL is better”. Hidden semantic assumptions have a description cost. A very short query requiring three undocumented business assumptions may lose to a slightly longer query backed by explicit schema evidence.

### Hypothesis
MDL-style regularization may reduce overengineered joins and accidental complexity on enterprise schemas.

---

## Idea M6 — Constraint-Solver Backed SQL Agent

### Borrowed principle
SAT/SMT solving separates declarative constraints from search.

### Text2SQL transfer
Let the LLM produce semantic constraints rather than directly committing to SQL:

```text
must_group_by = customer_id
must_exclude = refunds
join_cardinality(fact_orders, dim_customer) = many_to_one
metric = recognized_revenue
window = calendar_year(previous_year)
```

A symbolic layer rejects SQL candidates violating those constraints.

### Research value
This creates a clean separation between fuzzy language understanding and exact structural verification.

---

# 2. Statistics / Bayesian Inference / Information Theory

## Idea S1 — Bayesian Experimental Design for Tool Calls

### Borrowed principle
Bayesian experimental design chooses the next experiment to maximize expected information gain.

### Text2SQL transfer
Candidate meanings are latent hypotheses. Tool calls are experiments:

- inspect values
- inspect lineage
- sample duplicates
- query metadata
- ask user
- execute diagnostic SQL

Choose action `a` maximizing approximately:

`EIG(a) / cost(a)`

where EIG measures expected reduction in entropy over candidate semantic interpretations or SQL programs.

### Why this may matter
Most current agents use fixed tool orders or LLM intuition. A principled information-gain controller could use substantially fewer probes.

### Relation to active learning
This is active feature acquisition for a single query rather than active labeling across a dataset.

---

## Idea S2 — Query-by-Committee for SQL Evidence Acquisition

### Borrowed principle
Active learning's query-by-committee requests information where plausible models disagree most.

### Text2SQL transfer
Generate a committee of SQL hypotheses from diverse decoders/models/prompts. Do not majority-vote immediately. First identify the **structural disagreement**:

- join path
- aggregation function
- time column
- denominator
- null handling
- entity mapping

Then query the database or user specifically about that disagreement.

### Key distinction
Self-consistency uses agreement as an answer-selection mechanism. Here, disagreement determines **what evidence to acquire next**.

### Falsifiable prediction
Committee-driven probes solve more executable-but-disagreeing cases than either random probes or generic reflection at the same tool budget.

---

## Idea S3 — Sequential Probability Ratio Test for Agent Stopping

### Borrowed principle
Sequential hypothesis testing accumulates evidence until the likelihood ratio is strong enough to accept/reject a hypothesis.

### Text2SQL transfer
Stop the agent only when evidence for the leading SQL relative to alternatives crosses a confidence boundary.

Possible evidence increments:

- schema consistency
- execution success
- metamorphic tests
- candidate agreement
- verifier score
- business glossary match

### Contribution
A principled **adaptive compute policy**: easy queries stop early, hard queries keep probing.

### Evaluation
Plot semantic accuracy vs. mean tool calls and test whether sequential stopping dominates fixed-depth agents.

---

## Idea S4 — Conformal Selective Text2SQL

### Borrowed principle
Conformal prediction provides coverage-oriented uncertainty sets under exchangeability assumptions; selective prediction abstains when confidence is insufficient.

### Text2SQL transfer
Calibrate a nonconformity score using observable signals:

- candidate disagreement
- verifier margin
- schema retrieval entropy
- number of failed obligations
- execution instability under perturbations

At test time output one of:

```text
ACCEPT SQL
ASK CLARIFICATION
ABSTAIN / ESCALATE
```

### Important paper angle
The target is not merely calibration ECE. Measure **risk at coverage**: among automatically accepted SQLs, how low can semantic error be at 90%, 80%, 70% coverage?

### Enterprise value
A 95%-accurate system that knows which 20% not to answer can be more useful than a 97%-accurate system that confidently produces silent errors.

---

## Idea S5 — Information Bottleneck for Schema Context

### Borrowed principle
The information bottleneck seeks a compact representation preserving task-relevant information.

### Text2SQL transfer
Instead of retrieving top-k columns, learn a compressed **schema sufficient statistic** that minimizes irrelevant context while preserving all evidence required for the query.

Potential representation:

- selected schema subgraph
- values representative of entity mappings
- lineage snippets
- business definitions
- learned compressed relation summaries

### Strong variant
Train the selector against **semantic preservation tests**: if removing an item changes the correct program family, the item was not redundant.

---

# 3. NLP / Language Modeling

## Idea N1 — Self-Consistency as Version-Space Construction, Not Voting

### Borrowed principle
Self-consistency samples diverse reasoning paths and aggregates consistent answers.

### Text2SQL transfer
Do not use sampled SQLs merely for majority voting. Treat them as an empirical approximation to the posterior over programs.

Pipeline:

```text
sample N candidates
-> canonicalize AST
-> cluster by semantic structure
-> locate high-entropy decisions
-> probe only those decisions
-> collapse version space
```

### Novelty
Diversity becomes a **sensor for uncertainty**, not just an ensemble trick.

---

## Idea N2 — Noisy-Channel Text2SQL

### Borrowed principle
Classic NLP noisy-channel models combine a forward model with a reverse/explanatory model.

### Text2SQL transfer
Score candidate SQL using both:

`P(SQL | question, context)`

and a reverse model:

`P(question / semantic frame | SQL, schema, sampled execution behavior)`

A candidate should be rejected if the SQL cannot reconstruct the important semantic distinctions in the user's request.

### Example
If SQL counts orders but the original request says “customers”, a reverse semantic reconstruction should expose the mismatch even when execution succeeds.

### Research hook
Reverse explanation is structurally different from ordinary self-critique and may provide less correlated errors.

---

## Idea N3 — Natural Language Inference over SQL Claims

### Borrowed principle
NLI classifies whether evidence entails, contradicts, or is neutral to a claim.

### Text2SQL transfer
Translate SQL behavior into atomic claims, then perform entailment checks against user intent and schema evidence.

Example claims:

- “Each output row represents one customer.”
- “Revenue excludes refunds.”
- “The time filter refers to order creation time.”

Verifier labels each claim `ENTAILED | CONTRADICTED | UNKNOWN`.

### Termination rule
No critical claim may remain CONTRADICTED; UNKNOWN triggers a probe or clarification.

---

## Idea N4 — Contrastive Semantic Parsing

### Borrowed principle
Contrastive learning succeeds when carefully designed positive/negative transformations teach invariant and discriminative features.

### Text2SQL transfer
Construct hard contrastive pairs where one tiny semantic change must change SQL:

- before vs after
- customers vs orders
- gross vs net
- any vs all
- count vs count-distinct
- calendar vs fiscal year
- inner vs left inclusion semantics

Also create **invariance pairs** where superficial wording changes should not alter SQL semantics.

### Why attractive
Current synthetic Text2SQL data is large, but much of it may be easy. Contrastive examples directly target decision boundaries where agents fail.

---

## Idea N5 — Retrieval as Case-Based Reasoning over Failure Structures

### Borrowed principle
Case-based reasoning retrieves prior situations and adapts their solutions.

### Text2SQL transfer
Retrieve not nearest question-SQL pairs, but nearest **failure motifs**:

```text
many-to-many join explosion
ambiguous status encoding
slowly changing dimension
snapshot vs event table
fiscal calendar mismatch
bridge-table requirement
```

The retrieved case contains diagnosis + evidence probes + repair pattern, not final SQL.

### Benefit
This should transfer across schemas and reduce benchmark leakage concerns.

---

# 4. Computer Vision / Representation Learning

## Idea V1 — Semantic Augmentation Consistency

### Borrowed principle
Vision models learn invariances by enforcing consistent predictions under augmentations that preserve class identity.

### Text2SQL transfer
Create transformations that should preserve SQL semantics:

- rename tables/columns with consistent aliases
- reorder schema presentation
- paraphrase descriptions
- reorder independent joins
- permute irrelevant columns
- substitute equivalent date syntax

A robust agent should return semantically equivalent SQL.

Create transformations that should *change* semantics minimally and predictably:

- “top 10” -> “top 20”
- “2025” -> “2024”
- “customers” -> “orders”

### Research output
A **semantic equivariance benchmark** for Text2SQL agents.

### Why high leverage
It can diagnose shortcut reliance without requiring new human-labeled SQL for every perturbation.

---

## Idea V2 — Multi-View SQL Reasoning

### Borrowed principle
Multi-view vision combines complementary observations of the same latent scene.

### Text2SQL transfer
Construct multiple views of the same database task:

1. schema graph view
2. sample-value view
3. dbt/lineage view
4. natural-language documentation view
5. execution behavior view

Specialist modules produce beliefs that are fused only after independent processing.

### Hypothesis
Keeping views separate until late fusion reduces contamination from misleading evidence and makes ablations interpretable.

---

## Idea V3 — Coarse-to-Fine / Feature Pyramid SQL Generation

### Borrowed principle
Vision often solves recognition at multiple scales; coarse representations locate global structure before fine detail.

### Text2SQL transfer
Generate in levels:

```text
L0 intent frame
L1 relational skeleton
L2 join/filter plan
L3 aggregation/time semantics
L4 dialect-specific SQL
```

Errors can be localized to a scale. Repair should return to the earliest faulty level instead of rewriting the whole query.

### New metric
**repair locality**: how often can a failure be fixed without changing already-correct higher-level structure?

---

## Idea V4 — Test-Time Adaptation to a New Warehouse

### Borrowed principle
Test-time adaptation updates a model or small set of parameters using unlabeled target-domain signals.

### Text2SQL transfer
When deployed on a new enterprise warehouse, adapt lightweight components using self-supervised signals:

- schema naming co-occurrence
- PK/FK graph structure
- documentation consistency
- query logs if available
- SQL compiler feedback
- invariant/metamorphic checks

Do not fine-tune on confidential labels; adapt retrieval, ranking, embeddings, or normalization layers.

### Research question
Can source-free adaptation recover performance under schema/domain shift without labeled target SQL?

---

## Idea V5 — Adversarial Occlusion for Evidence Attribution

### Borrowed principle
Occlusion tests in vision remove image regions to see which evidence drives a prediction.

### Text2SQL transfer
Remove or mask pieces of context and observe candidate stability:

- remove a schema description
- hide a sample value
- remove one retrieved document
- mask one join edge

If a supposedly critical decision is unchanged, the cited evidence may be post-hoc. If a tiny irrelevant context change flips the query, the agent is brittle.

### Contribution
An **evidence faithfulness audit** for SQL agents.

---

# 5. Reinforcement Learning / Search / Decision Making

## Idea R1 — MCTS over SQL Agent Trajectories

### Borrowed principle
Monte Carlo Tree Search balances exploration of uncertain branches with exploitation of promising branches.

### Text2SQL transfer
Nodes are agent states; edges are actions such as:

- retrieve schema
- inspect values
- choose join
- generate candidate
- run probe
- ask user
- repair

Rollout reward includes semantic correctness proxy, evidence completeness, cost, and latency.

### Crucial modification
Do not let the same LLM both rollout and score. Use deterministic constraints + independent verifier signals.

### Falsifier
If a simpler best-first disagreement-driven search achieves the same accuracy/cost Pareto frontier, MCTS is unnecessary.

---

## Idea R2 — Contextual Bandit Tool Router

### Borrowed principle
Bandits choose among actions while learning their expected reward under context.

### Text2SQL transfer
Learn a cheap policy deciding which tool to call next:

`schema search | value search | lineage | EXPLAIN | execute | verifier | clarification`

Reward is downstream error reduction minus cost.

### Why practical
A bandit router is easier to train and deploy than full RL and may capture most of the economic value.

---

## Idea R3 — Hierarchical Agent / Options Framework

### Borrowed principle
Hierarchical RL defines reusable temporally extended actions (“options”).

### Text2SQL transfer
Create reusable macro-skills:

- resolve entity mapping
- validate join cardinality
- resolve temporal semantics
- validate denominator
- debug execution error
- optimize expensive query

The top-level controller selects an option; each option has its own termination condition and evidence contract.

### Research benefit
This gives a principled alternative to an arbitrary multi-agent collection.

---

## Idea R4 — Model Predictive Control for SQL Agents

### Borrowed principle
MPC repeatedly plans over a horizon, executes only the first action, observes new state, then replans.

### Text2SQL transfer
Rather than committing to a full long chain, repeatedly plan the next 2–4 tool calls, execute one, and update based on evidence.

### Why useful
Database observations can invalidate the original plan. Receding-horizon planning should be more robust than a one-shot agent plan while cheaper than unrestricted tree search.

---

# 6. Causal Inference

## Idea C1 — Causal Semantics for Metric Selection

### Borrowed principle
Causal inference distinguishes observational association from interventions and makes assumptions explicit.

### Text2SQL transfer
Many analytics questions are accidentally treated as simple aggregation but imply causal or temporal semantics:

- “impact of campaign”
- “customers retained because of discount”
- “effect of outage on conversion”

The SQL agent should detect causal language and **refuse to collapse the task into naive aggregation** without assumptions.

### Contribution
A causal-intent classifier + safe handoff protocol that distinguishes descriptive SQL from causal analysis.

### Safety/reliability angle
The best SQL can still answer the wrong scientific question.

---

## Idea C2 — Counterfactual Database Perturbation

### Borrowed principle
Counterfactual reasoning asks whether conclusions would change under controlled interventions.

### Text2SQL transfer
Create synthetic or sandboxed perturbations:

- duplicate rows
- inject nulls
- move events across date boundary
- remove dimension matches
- alter category frequencies

Observe whether candidate SQL changes in the theoretically expected way.

### Example
If the query claims to count unique customers, duplicating order rows should not change the answer.

### Connection
This generalizes metamorphic testing into a causal perturbation framework.

---

## Idea C3 — Invariance across Environments

### Borrowed principle
Invariant prediction seeks relationships that remain stable across environments.

### Text2SQL transfer
Correct semantic rules should often transfer across:

- database snapshots
- shards/regions
- schema aliases
- sampled subsets

Wrong shortcuts often fail invariance tests.

### Research direction
Score candidate SQL by stability of semantic properties across controlled environments rather than only one execution result.

---

# 7. Software Testing / Security

## Idea T1 — Property-Based SQL Testing

### Borrowed principle
Property-based testing generates many inputs and checks general invariants instead of hand-writing individual test cases.

### Text2SQL transfer
Generate query-specific properties automatically:

- monotonicity under filter relaxation
- conservation across exhaustive partitions
- uniqueness after grouping
- referential integrity expectations
- bounds on ratios and counts

Then fuzz the database/query conditions and search for violations.

### Key advantage
No gold SQL is required for many checks.

---

## Idea T2 — Mutation Testing for SQL Verifiers

### Borrowed principle
Mutation testing measures test quality by injecting small bugs and checking whether tests catch them.

### Text2SQL transfer
Generate semantically plausible SQL mutants:

- COUNT -> COUNT DISTINCT
- >= -> >
- INNER -> LEFT
- order_date -> ship_date
- remove one predicate
- swap denominator
- move filter before/after aggregation

A verifier is good only if it “kills” a high fraction of realistic mutants.

### Strong paper angle
Instead of asking whether a verifier correlates with accuracy, directly evaluate its **semantic bug detection power**.

### Metric
Mutation score stratified by bug family.

---

## Idea T3 — SQL Fuzzing for Silent Error Discovery

### Borrowed principle
Fuzzers explore inputs that trigger crashes or unexpected behavior.

### Text2SQL transfer
Fuzz semantic inputs rather than parser bytes:

- adversarial schema names
- near-duplicate columns
- misleading descriptions
- null-heavy data
- many-to-many bridges
- timezone edge cases
- fiscal calendar shifts

Goal: discover conditions that make an apparently reliable agent fail silently.

### Research output
A benchmark generator for adversarial enterprise SQL environments.

---

## Idea T4 — Delta Debugging for Agent Failures

### Borrowed principle
Delta debugging minimizes a failing input to the smallest failure-inducing subset.

### Text2SQL transfer
When an agent fails on a large schema/context, automatically shrink:

- schema tables
- columns
- documentation snippets
- data rows
- conversation turns

until the minimal configuration still causes the wrong SQL.

### Why valuable
This turns opaque benchmark failures into interpretable mechanistic cases and produces high-quality training examples.

---

# 8. Learning Theory / Ensemble Methods

## Idea L1 — Boosting over Failure Modes

### Borrowed principle
Boosting repeatedly focuses learning capacity on examples the current ensemble gets wrong.

### Text2SQL transfer
Maintain a failure taxonomy and dynamically upweight under-solved semantic motifs:

- join explosion
- hidden denominator
- temporal mismatch
- ambiguous entity
- SCD handling
- nested aggregation

Train or prompt specialist verifiers for the residual error distribution rather than uniformly scaling the model.

### Interesting architecture
A sequence of small critics where each critic is trained on the residual failures of previous critics.

---

## Idea L2 — Curriculum over Semantic Distance

### Borrowed principle
Curriculum learning orders examples from easier to harder.

### Text2SQL transfer
Define semantic distance between correct SQL and minimal wrong mutants. Start with one-slot differences, then compose multiple interacting ambiguities.

Example curriculum:

```text
1 semantic choice wrong
-> 2 interacting choices
-> hidden business rule + join issue
-> multi-stage repository task
```

### Why useful
Could make verifier training substantially more sample-efficient than random synthetic generation.

---

# 9. High-Risk / High-Reward Composite Ideas

## Composite X1 — Scientific-Method SQL Agent

Treat every SQL task as a miniature scientific investigation:

```text
hypothesis generation
-> experiment design
-> observation
-> hypothesis elimination
-> replication / invariance checks
-> confidence calibration
-> report + evidence
```

This combines CEGIS, Bayesian experimental design, sequential testing, and proof-carrying output.

### Potential claim
“Text2SQL improves when the agent is optimized as an experimental scientist rather than a code generator.”

### Minimal prototype
No training required. Implement:

1. diverse candidate generation
2. structural disagreement extraction
3. probe generator
4. independent verifier
5. sequential evidence stopping

This is currently my strongest cross-domain direction.

---

## Composite X2 — Proof-Carrying MCTS SQL Agent

MCTS explores plans, but a leaf is terminal only if all semantic obligations are certified.

Reward:

`verified semantic evidence - tool cost - unresolved obligations - execution cost`

This prevents the classic failure mode where tree search optimizes a weak self-evaluation score.

### Risk
Potentially too expensive. Must beat best-first or disagreement-first search under equal tool budget.

---

## Composite X3 — Conformal Active SQL Agent

Combine selective prediction with active information acquisition:

1. estimate uncertainty set over SQL semantic clusters
2. if singleton and calibrated -> answer
3. otherwise select highest-information probe
4. update uncertainty set
5. if budget exhausted -> ask/abstain

### Why elegant
It unifies three deployment concerns in one framework:

- accuracy
- interaction cost
- abstention reliability

---

## Composite X4 — Semantic Mutation Gym

Build a training/evaluation environment from mutation testing + contrastive learning + curriculum.

For each correct or high-quality SQL:

1. generate typed semantic mutants
2. execute both
3. retain hard mutants that execute successfully and look plausible
4. train verifiers to identify the bug family
5. progressively compose mutations

### Possible contribution
A general-purpose **hard-negative factory** for Text2SQL agent verification.

This could be more reusable than a new end-to-end agent architecture.

---

## Composite X5 — Warehouse Test-Time Adaptation with Failure Memory

When the agent enters a new warehouse:

1. build schema graph
2. run cheap self-supervised probes
3. adapt retriever/ranker
4. accumulate failure motifs
5. update environment-specific memory
6. never store benchmark-specific final answers

### Hypothesis
Most enterprise value comes from **within-warehouse continual adaptation**, not cross-database zero-shot performance.

---

# 10. Ranking: which transfers look most publishable?

Scoring is qualitative: 5 = strongest.

| Idea | Novelty | Tractability | Scientific clarity | Likely impact | Main risk |
|---|---:|---:|---:|---:|---|
| Scientific-Method SQL Agent | 5 | 4 | 5 | 5 | components may each appear incremental unless unified sharply |
| Semantic Mutation Gym | 4 | 5 | 5 | 5 | benchmark/data contribution must prove realism |
| Conformal Active SQL Agent | 5 | 3 | 5 | 5 | calibration assumptions under distribution shift |
| Query-by-Committee Evidence | 4 | 5 | 5 | 4 | could look like self-consistency unless active probing is central |
| Proof-Carrying SQL | 5 | 4 | 5 | 5 | proof obligations may be only heuristic |
| Bayesian Tool Selection | 5 | 3 | 5 | 4 | estimating EIG cheaply is difficult |
| Mutation Testing Verifier | 4 | 5 | 5 | 5 | requires realistic mutation taxonomy |
| Semantic Equivariance Benchmark | 4 | 5 | 5 | 4 | needs careful validity of transformations |
| Test-Time Warehouse Adaptation | 5 | 3 | 4 | 5 | harder benchmark setup |
| MCTS SQL Agent | 3 | 3 | 4 | 4 | cost; easier search may match it |
| MDL SQL Selection | 4 | 4 | 3 | 3 | description-length design may feel arbitrary |
| Causal-intent SQL Guard | 4 | 5 | 5 | 4 | narrower than core Text2SQL generation |

---

# 11. Recommended next three experiments

## Experiment A — Semantic Mutation Gym pilot

Build 8–12 mutation operators and create 500–2,000 executable hard negatives from a public Text2SQL set.

Question:

> Can generic LLM critique, execution testing, and the proposed evidence verifier actually detect realistic semantic mutants?

This is cheap and generates diagnostic data for every later idea.

## Experiment B — Query-by-Committee + active probe selection

On samples with multiple executable candidates:

1. cluster candidate SQL by semantic structure
2. identify the decision with maximum disagreement
3. generate one targeted database probe
4. compare against random probe, generic reflection, and majority vote

Question:

> Does disagreement-driven evidence acquisition select the correct semantic cluster more efficiently?

## Experiment C — Sequential / conformal stopping

Use signals from A/B to train or calibrate a stopping score. Compare fixed 1/3/5-step agents against adaptive stopping.

Question:

> Can we reduce tool cost while holding accepted-query semantic risk constant?

If A, B, and C all work, they compose naturally into a single “scientific-method SQL agent”.

---

# 12. Reference anchors from the borrowed domains

These are conceptual anchors, not claims that the original papers address Text2SQL.

- Solar-Lezama et al. / Sketch & CEGIS lineage — counterexample-guided program synthesis.
- Kocsis & Szepesvári (2006), *Bandit Based Monte-Carlo Planning* — UCT / MCTS.
- Settles (2009), *Active Learning Literature Survey* — uncertainty sampling, query-by-committee, expected error reduction.
- Tishby, Pereira & Bialek, *The Information Bottleneck Method* — compress context while preserving task-relevant information.
- Wang et al. (2022/ICLR 2023), *Self-Consistency Improves Chain of Thought Reasoning in Language Models* — diverse reasoning paths and consistency.
- Chen et al. (2020), *A Simple Framework for Contrastive Learning of Visual Representations (SimCLR)* — augmentation design and contrastive representation learning.
- Wang et al. (2020), *Tent: Fully Test-time Adaptation by Entropy Minimization* — source-free test-time adaptation.
- Classical sequential hypothesis testing / SPRT — evidence accumulation and adaptive stopping.
- Conformal prediction — calibrated prediction sets / selective risk control.
- Property-based testing, mutation testing, fuzzing, and delta debugging — systematic verification and failure minimization.

## Closing observation

The recurring lesson from other fields is that **generation is rarely the whole algorithm**. Strong systems search, test, compress, calibrate, adapt, and certify. Text2SQL is unusually well suited to this transfer because SQL is executable and databases expose rich diagnostic actions. That gives us something many NLP tasks do not have: an environment in which an agent can actively run experiments against its own hypotheses.
