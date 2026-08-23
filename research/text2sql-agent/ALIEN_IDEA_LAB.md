# Alien Idea Lab: Deliberately Unconventional Text2SQL × Agent Research Directions

> Goal: escape the local optimum of planner / critic / verifier / retrieval / self-consistency. This memo imports mechanisms from fields that are rarely used as the *main abstraction* for Text2SQL. Many ideas are intentionally high-risk. The criterion is not immediate practicality; it is whether the transfer exposes a new variable that could support a falsifiable research paper.

## 0. Rule of this memo

Do not ask: “How can we add this technique to an SQL agent?”

Ask instead:

> “If this other field were forced to solve Text2SQL, what object would it optimize, what state would it maintain, and what would it consider evidence?”

That change of ontology is where genuinely different ideas tend to appear.

---

# I. Topology, Geometry, and Algebra

## 1. Topological Text2SQL: preserve semantic holes, not token form

### Inspiration
Topology studies properties preserved under continuous deformation.

### Transfer
Treat paraphrases, harmless schema renamings, join re-orderings, and equivalent decompositions as deformations of the same semantic object. Instead of training/querying over strings, define a *semantic topology* over query programs.

The interesting object is not one SQL program but its equivalence class under semantics-preserving transformations.

### Research question
Can a model learn representations in which equivalent SQL programs remain in the same connected component, while subtle semantic changes—wrong grain, wrong denominator, wrong temporal anchor—cross a topological boundary?

### Possible method
Build a graph whose nodes are SQL programs and edges are typed transformations. Estimate persistent connected components under progressively stronger semantic tests. A candidate that sits in an unstable component is suspicious.

### Wild hypothesis
Semantic correctness may be better predicted by *component stability* than by model confidence.

---

## 2. Persistent Homology for ambiguity structure

### Inspiration
Persistent homology tracks structures that survive across scales.

### Transfer
Generate many semantic hypotheses under different retrieval budgets, schema scopes, temperatures, and model families. Cluster them by behavior. Some disagreements disappear quickly; others persist regardless of scale.

Call persistent disagreements **semantic holes**.

### Use
- decide when clarification is unavoidable
- prioritize which ambiguity to resolve
- identify benchmark questions with intrinsically unstable semantics

### Paper angle
“Persistent Ambiguity in Text2SQL”: quantify which semantic uncertainties survive more context, more compute, and more agents.

---

## 3. Optimal Transport between language intent and database structure

### Inspiration
Optimal transport moves mass between distributions with minimal cost.

### Transfer
Represent question concepts as semantic mass and schema entities as another distribution. Solve a transport problem where cost reflects concept mismatch, documentation distance, lineage distance, and business-semantic mismatch.

Instead of hard schema linking, obtain a soft transport plan:

`question concepts -> tables / columns / metrics / values`

### Why different
Most schema linking is retrieval or classification. OT naturally allows one concept to distribute across multiple schema objects and can expose ambiguous mass that has not committed.

### Test
Does transport entropy predict downstream SQL failure better than top-k retrieval score?

---

## 4. Category-theoretic query composition

### Inspiration
Category theory focuses on compositional mappings and structure-preserving transformations.

### Transfer
Treat semantic operations as morphisms:

- filter
- aggregate
- group
- map entity
- change grain
- join relation

Complex analytics questions become compositions of typed morphisms rather than monolithic SQL strings.

### Research angle
Learn reusable compositional laws and test whether they generalize across schemas:

`customer -> orders -> revenue -> rank`

should preserve structure even when physical schemas differ radically.

### Extreme variant
Use functor-like mappings from a business semantic category into different warehouse schemas/dialects. The same intent compiles into multiple data systems while preserving composition laws.

---

## 5. Sheaf-style local-to-global SQL reasoning

### Inspiration
Sheaf theory formalizes when locally consistent information can be glued into a globally consistent object.

### Transfer
Different specialists solve local pieces:

- temporal semantics
- entity mapping
- metric definition
- join path
- output grain

Each may be locally correct. The central question becomes: can these local assignments be glued into one globally consistent SQL interpretation?

### New failure signal
A query can have no globally consistent solution even though every pairwise local decision looks reasonable.

### Practical approximation
Build an overlap-consistency graph. Specialists must agree on shared variables. Detect obstruction cycles where pairwise agreements cannot all hold simultaneously.

---

## 6. Group theory for symmetry-aware SQL generation

### Inspiration
Groups describe transformations that preserve structure.

### Transfer
Identify symmetry groups of a task:

- renaming equivalent IDs
- permuting dimension order
- equivalent join association
- swapping synonymous business aliases

Require the agent’s behavior to respect these symmetries.

### Novel metric
Symmetry violation rate: how often does a harmless transformation cause semantic drift?

### Strong use
Generate symmetry-based stress tests without new labels.

---

# II. Control Theory, Dynamical Systems, and Robotics

## 7. SQL Agent as a feedback controller

### Inspiration
Control theory does not “plan once”; it observes the system and continuously corrects control actions.

### Transfer
State = belief about user intent + schema semantics + database behavior.

Action = retrieve, probe, ask, generate, repair.

Observation = query result statistics, schema evidence, user response, execution errors.

The goal is to drive *semantic error* toward zero while minimizing control energy.

### Contribution
Study controller stability instead of raw final accuracy.

A bad agent oscillates:

`join A -> join B -> join A -> join B`

or over-corrects after weak evidence. This can be measured as trajectory instability.

---

## 8. Model Predictive Control for SQL reasoning

### Inspiration
MPC repeatedly plans over a finite horizon, executes the first action, observes, then replans.

### Transfer
At each step, simulate short future trajectories of possible tool calls. Choose the first action with lowest expected future semantic risk + cost. Replan after every new observation.

### Difference from generic tree search
The object is a rolling *closed-loop controller* with explicit horizon and cost, not an open-ended reasoning tree.

### Experiment
Compare 1-step greedy, fixed pipeline, and horizon-3 MPC under the same tool budget.

---

## 9. Lyapunov functions for agent termination

### Inspiration
A Lyapunov function proves that a dynamical system is converging toward a stable state.

### Transfer
Define a scalar semantic-energy function combining:

- unresolved ambiguities
- candidate disagreement
- violated invariants
- missing evidence
- uncertainty

Require each reasoning step to decrease the energy in expectation.

### Why useful
Agent loops often consume tokens without measurable progress. A Lyapunov-like progress certificate could detect useless trajectories early.

### Research question
Can a learned progress function predict whether another reasoning step is worth taking?

---

## 10. SLAM for database understanding

### Inspiration
Robots perform Simultaneous Localization and Mapping: infer where they are while building a map of an unknown environment.

### Transfer
A SQL agent entering a new warehouse has the same chicken-and-egg problem:

- it needs the schema map to answer queries
- but it learns the useful schema map by answering queries

Build a **semantic warehouse map** over time:

- entities
- relationships
- trustworthy join paths
- metric definitions
- dangerous regions
- recurring business aliases

### New framing
Text2SQL becomes **Simultaneous Querying and Warehouse Mapping (SQWM)**.

### Evaluation
After N tasks, how much faster and safer does the agent become on task N+1?

---

## 11. Loop closure for warehouse memory

### Inspiration
In SLAM, loop closure recognizes a previously visited place and corrects accumulated map drift.

### Transfer
When a new query revisits a known business concept, compare current interpretation with historical warehouse memory.

If they disagree, trigger semantic loop closure:

- verify whether schema changed
- whether old memory was wrong
- whether business definition drifted

### High-value enterprise angle
Detect silent semantic drift over months of repeated use.

---

## 12. Active perception instead of active retrieval

### Inspiration
Robotics distinguishes passive sensing from actions chosen specifically to reduce perceptual uncertainty.

### Transfer
A diagnostic SQL query is not merely information retrieval—it changes what the agent can infer. Design actions whose primary purpose is to make hidden semantics observable.

Example: instead of sampling random rows, query rare boundary cases likely to distinguish two join hypotheses.

### Thesis
The best SQL agent behaves like an embodied robot performing active perception inside a database.

---

# III. Causal Science and Counterfactual Reasoning

## 13. Structural causal model over query intent

### Inspiration
Causal inference separates correlation from intervention and models variables with structural equations.

### Transfer
Construct an SCM over semantic decisions:

`business concept -> entity set -> join path -> grain -> aggregation -> result`

Then ask counterfactual questions:

- if the join path changed but grain stayed fixed, what should happen?
- if “active” definition changed, which clauses should change and which should remain invariant?

### Use
Diagnose *where* an incorrect answer originates, not just that it is wrong.

---

## 14. Do-calculus-inspired semantic interventions

### Transfer
Rather than perturbing arbitrary tokens, intervene on one semantic variable at a time:

`do(time_window = 2024)`
`do(metric = net_revenue)`
`do(entity = orders)`

Check whether the generated SQL changes only in the causal descendants of that variable.

### New benchmark
**Interventional Text2SQL Consistency**: whether SQL responds correctly to controlled semantic interventions.

---

## 15. Instrumental variables for hidden business semantics

### Inspiration
Instrumental variables identify causal effects when a latent confounder cannot be observed directly.

### Wild transfer
Sometimes a business metric definition is undocumented. But related dashboards, tests, or downstream models may act as instruments that constrain the latent definition.

Use external artifacts to infer an unobserved semantic variable without directly observing its canonical definition.

### Research question
Can indirect repository evidence recover hidden metric semantics better than direct language-model guessing?

---

## 16. Causal discovery over warehouse lineage

### Inspiration
Causal discovery infers hidden graph structure from observations/interventions.

### Transfer
When PK/FK metadata is incomplete, actively infer likely semantic relationships from:

- co-occurrence
- cardinality patterns
- temporal ordering
- lineage
- intervention-like query probes

The goal is not true causality but causal-discovery-style *structure learning* for hidden relational semantics.

---

# IV. Logic, Theorem Proving, and Knowledge Revision

## 17. Abductive Text2SQL

### Inspiration
Abduction finds the best explanation for observations.

### Transfer
Instead of “generate SQL from question,” invert the problem:

> What hidden semantic assumptions would make this SQL/result a plausible explanation of the user’s request?

Generate assumptions explicitly, then reject candidates requiring too many unsupported assumptions.

### Score
`fit to question - unsupported assumption cost`

### Benefit
Makes hidden business assumptions visible.

---

## 18. Belief revision rather than prompt accumulation

### Inspiration
AGM belief revision studies how a rational system should update beliefs when new evidence contradicts old beliefs.

### Transfer
Agents currently accumulate context without principled contradiction handling. Maintain a belief base:

- `active_customer := status='A'`
- `revenue := net_amount`

When evidence conflicts, revise the minimum necessary subset rather than appending both claims.

### Paper angle
“Belief Revision for Long-Lived Database Agents.”

---

## 19. Paraconsistent SQL reasoning

### Inspiration
Paraconsistent logic allows reasoning in the presence of contradictions without explosion.

### Transfer
Enterprise metadata is often contradictory:

- docs say one thing
- dbt code says another
- dashboard uses a third definition

Do not force immediate resolution. Track contradictory support and allow local reasoning to continue while marking affected conclusions as contested.

### Output
Answer may include a *semantic conflict certificate* rather than hallucinating one canonical definition.

---

## 20. Argumentation frameworks for semantic decisions

### Inspiration
Abstract argumentation models claims, attacks, and defenses.

### Transfer
Build an argument graph:

- claim: use `order_date`
- support: metric documentation
- attack: dashboard uses `ship_date`
- defense: dashboard is deprecated

Final SQL decisions are accepted only if their supporting arguments survive attacks under a chosen semantics.

### Different from debate
The artifact is a persistent graph of reasons and conflicts, not conversational persuasion.

---

## 21. SAT/SMT-backed SQL semantic completion

### Inspiration
SAT/SMT solvers search discrete spaces subject to hard constraints.

### Transfer
LLM proposes a partial semantic sketch with holes:

- unknown join path
- unknown date column
- unknown aggregation

Translate available evidence into constraints and let a solver enumerate satisfying completions.

### Hybrid architecture
LLM handles soft linguistic interpretation; solver handles combinatorial consistency.

### Strong experiment
Does constrained completion reduce hallucinated schema choices on very large schemas?

---

## 22. Minimal unsatisfiable cores as explanations

### Inspiration
SMT solvers can return a minimal set of mutually inconsistent constraints.

### Transfer
When no SQL interpretation satisfies all user requirements + schema facts + business rules, extract a minimal semantic conflict set.

Example:

- “unique customers”
- “one row per order”
- “return customer-level revenue”

may imply contradictory grain requirements.

### Product value
Ask a *precise* clarification derived from the conflict core.

---

# V. Ecology, Evolution, and Population Dynamics

## 23. Ecological niches for specialist agents

### Inspiration
Species coexist when they occupy different niches rather than competing identically.

### Transfer
Train specialists with sharply separated ecological niches:

- temporal queries
- graph-like joins
- financial metrics
- cohort analysis
- slowly changing dimensions

Use a router that measures niche fit.

### Why different from mixture-of-experts
Study *coexistence and specialization pressure* over a changing workload, not a fixed gating network.

---

## 24. Predator-prey dynamics for generator-verifier systems

### Inspiration
Predator and prey populations co-evolve.

### Transfer
Generators produce increasingly subtle semantic mistakes; verifiers adapt to catch them. But if verifier strength grows too quickly, generator diversity collapses; if too weak, silent errors proliferate.

### Research object
Study the dynamic equilibrium of bug-generation and bug-detection populations.

### Metric
Semantic bug diversity over training time, not just mean accuracy.

---

## 25. Ecological succession for warehouse adaptation

### Inspiration
Ecosystems change from pioneer species to stable mature communities.

### Transfer
A new warehouse agent should not use the same architecture on day 1 and day 100.

Early phase:
- broad exploration
- high uncertainty
- expensive probes

Mature phase:
- stable semantic map
- cached relationships
- cheap specialized routines

### Thesis
Continual SQL agents should undergo *architectural succession* as environment knowledge accumulates.

---

## 26. Epidemiology of semantic errors

### Inspiration
Epidemiology studies how failures propagate through populations and networks.

### Transfer
A wrong business definition in one memory item can infect many future SQL queries.

Model semantic bugs as contagion over:

- shared memories
- templates
- schema links
- downstream dashboards
- agent-generated documentation

### Key variable
Reproduction number `R_semantic`: expected number of future answers corrupted by one bad semantic fact.

### New priority rule
Fix high-R errors first even if they are rare.

---

## 27. Vaccination against semantic bugs

### Transfer
Once a high-risk failure pattern is found, create a small “vaccine”:

- invariant
- probe
- negative example
- schema warning

that is injected into future reasoning only when the relevant risk signature appears.

### Goal
Avoid globally bloating prompts while still building immunity.

---

# VI. Neuroscience and Cognitive Architectures

## 28. Hippocampal replay for SQL agents

### Inspiration
Brains replay experiences offline, potentially consolidating memory.

### Transfer
During idle time, replay difficult warehouse trajectories and compress them into:

- reusable schema motifs
- failure warnings
- semantic aliases
- join-path heuristics

### Distinction
Online agent behavior stays simple; expensive consolidation happens offline.

### Research question
Does offline replay improve future query performance more safely than online self-modification?

---

## 29. Complementary learning systems

### Inspiration
Cognitive models distinguish fast episodic memory from slow structured learning.

### Transfer
Maintain two memories:

1. episodic: exact previous incidents and query trajectories
2. semantic: slowly consolidated warehouse rules

Never immediately promote one successful query into a permanent rule.

### Why valuable
Reduces catastrophic overgeneralization from one-off accidents.

---

## 30. Predictive coding hierarchy for SQL

### Inspiration
Hierarchical predictive coding sends predictions downward and errors upward.

### Transfer
Layers:

- business intent
- semantic plan
- relational plan
- SQL AST
- execution statistics

Each layer predicts constraints on the layer below. Mismatches create prediction-error signals sent upward to revise the smallest responsible abstraction.

### Benefit
Repair at the right level. A semantic error should not be patched as a syntax error.

---

## 31. Neural synchronization as consensus signal

### Wild inspiration
Distributed neural populations often coordinate through synchronization.

### Transfer
Instead of averaging agent answers, measure *timing/order of semantic commitments* across independent agents. Decisions that synchronize early may represent easy consensus; late or unstable synchronization marks fragile semantics.

### Speculative metric
Commitment synchronization entropy.

---

# VII. Thermodynamics and Statistical Physics

## 32. Free-energy-minimizing SQL agent

### Inspiration
Variational free energy combines model fit and complexity.

### Transfer
Score semantic hypotheses with two terms:

- evidence mismatch
- complexity / unsupported assumption cost

Prefer explanations that fit observations without inventing excessive hidden semantics.

### Potential benefit
A principled anti-hallucination objective for ambiguous schema interpretation.

---

## 33. Phase transitions in reasoning budget

### Inspiration
Physical systems can change qualitatively when a control variable crosses a critical point.

### Transfer
Measure accuracy as interaction budget increases. Some tasks may show abrupt transitions:

- below 2 probes: hopeless
- at 3 probes: ambiguity resolves
- beyond 4: no gain

### Research question
Can we predict a task’s *critical reasoning budget* from early signals?

### Product use
Allocate compute near the phase transition instead of uniformly.

---

## 34. Entropy production as wasted reasoning

### Transfer
Define irreversible reasoning waste:

- repeated probes
- discarded candidate generations
- redundant retrieval
- oscillating repairs

Estimate “semantic entropy production” per solved task.

### Goal
Optimize not just cost but *irreversible waste*.

---

## 35. Renormalization: solve SQL at multiple scales

### Inspiration
Renormalization studies a system by coarse-graining and moving between scales.

### Transfer
Reason at progressively finer schema scales:

1. business domain
2. entity graph
3. table graph
4. columns
5. values

Do not expose fine-grained schema until the coarse interpretation is stable.

### Hypothesis
Multi-scale reasoning reduces distraction in 1000+ column schemas.

---

# VIII. Signal Processing and Inverse Problems

## 36. Compressed sensing for schema discovery

### Inspiration
Sparse signals can be reconstructed from surprisingly few measurements.

### Transfer
Assume only a tiny subset of a huge warehouse matters for each query. Rather than retrieve many columns, design a small set of diagnostic measurements that reconstruct the relevant semantic support.

### Research question
How few schema/value probes are sufficient to recover the true relevant subgraph?

### Metric
Support recovery vs probe budget.

---

## 37. Spectral graph methods for schema reasoning

### Inspiration
Graph spectra reveal global structural properties.

### Transfer
Represent schema + lineage + co-query usage as a graph. Use spectral structure to discover:

- semantic communities
- bridge tables
- hubs
- suspicious cross-domain joins

### Different angle
Schema linking can exploit global graph modes rather than local embedding similarity.

---

## 38. Frequency-domain query analysis

### Wild transfer
Represent workload behavior over time and detect recurring semantic patterns at different frequencies:

- daily operational questions
- monthly finance closes
- quarterly business metrics

Use temporal frequency signatures to disambiguate which metric/table family a user likely means.

### Enterprise angle
Context from workload periodicity becomes part of intent inference.

---

## 39. Blind source separation for mixed business concepts

### Inspiration
ICA separates latent sources from observed mixtures.

### Transfer
User questions often mix concepts such as bookings, refunds, recognized revenue, and cash collected. Historical queries/results form mixtures of latent business definitions.

Infer reusable latent semantic factors, then map new questions onto them.

---

# IX. Operations Research and Scheduling

## 40. Constraint programming for evidence scheduling

### Transfer
A hard SQL question may have dozens of possible tool calls with dependencies. Formulate evidence acquisition as a resource-constrained scheduling problem:

- some probes can run in parallel
- some require prior schema resolution
- warehouse scans have budgets
- user clarification has latency cost

Optimize makespan under a target confidence threshold.

### New metric
Time-to-trustworthy-answer rather than token count.

---

## 41. Queueing theory for multi-query agents

### Inspiration
Production agents receive many simultaneous analytics requests.

### Transfer
Reasoning budget should depend on queue state. A query that could consume 40 seconds of verification may be deferred or degraded when the system is overloaded.

### Paper angle
Text2SQL quality–latency control under realistic workload queues.

---

## 42. Real-options theory for delayed commitment

### Inspiration
In finance, an option has value because commitment can be delayed until uncertainty resolves.

### Transfer
Treat schema/semantic commitment as an irreversible investment. Maintaining two candidate interpretations for one more step can have positive option value.

### Research question
When is “do not commit yet” the optimal SQL-agent action?

### Metric
Value of delayed commitment under fixed cost.

---

# X. Human Factors, Law, and Institutional Reasoning

## 43. Legal precedent retrieval for business semantics

### Inspiration
Law resolves ambiguous rules partly through precedent.

### Transfer
Enterprise warehouses also have precedent:

- prior accepted dashboards
- audited reports
- historical analyst queries
- incident resolutions

Instead of nearest-neighbor SQL retrieval, retrieve *precedents that establish a semantic rule*.

### Example
A prior finance reconciliation may establish that “revenue” means recognized net revenue in a particular reporting context.

---

## 44. Burden of proof for risky SQL decisions

### Inspiration
Legal systems assign different burdens of proof.

### Transfer
Not all SQL decisions need the same evidence threshold.

- harmless alias choice: preponderance
- financial denominator: clear-and-convincing
- destructive warehouse action: beyond-reasonable-doubt equivalent

### Contribution
Risk-sensitive evidence thresholds instead of one global confidence cutoff.

---

## 45. Appeals process for SQL agents

### Transfer
When a reviewer rejects SQL, do not merely regenerate. Create an appeal path:

1. generator states contested point
2. reviewer cites evidence
3. independent adjudicator sees only the dispute
4. decision becomes precedent or remains unresolved

### Why potentially useful
Separates local disputes from full-query regeneration and creates structured training data.

---

## 46. Organizational memory as institutional knowledge

### Transfer
Some semantic truth belongs neither to schema nor model weights but to organizational process. Build an institutional layer that tracks:

- who owns a metric
- which source is authoritative
- freshness / deprecation
- exceptions
- governance history

### Thesis
Enterprise Text2SQL is partly a governance problem, not merely semantic parsing.

---

# XI. Music, Language, and Sequence Structure

## 47. Harmonic analysis of SQL plans

### Inspiration
Music has tension, resolution, motifs, and hierarchical structure.

### Transfer
Treat common analytics constructions as motifs:

- cohort motif
- funnel motif
- retention motif
- ranking motif
- period-over-period motif

A query is composed by transforming and combining motifs.

### Research direction
Learn motif-level program grammars that generalize better than token generation.

---

## 48. Counterpoint as multi-view reasoning

### Inspiration
Independent melodic lines can remain individually coherent while forming a coherent whole.

### Transfer
Run independent semantic interpretations constrained to be complementary, not identical:

- entity-centric voice
- metric-centric voice
- temporal voice
- data-quality voice

The final SQL must harmonize all voices.

### Difference from multi-agent debate
Agents are rewarded for orthogonality, not winning.

---

## 49. Prosody-like signals from user wording

### Inspiration
Prosody carries meaning beyond literal lexical content.

### Transfer
In interactive SQL systems, punctuation, emphasis, correction patterns, and phrase order may reveal which ambiguity matters most.

Example:

“revenue by *customer*, not by order”

is a grain correction signal stronger than generic lexical similarity.

### Research angle
Model conversational repair acts as a distinct semantic channel.

---

# XII. Swarm Intelligence and Collective Search

## 50. Ant-colony schema search

### Inspiration
Ant colony optimization reinforces useful paths through pheromone trails.

### Transfer
Each successful query deposits virtual pheromone on schema/join paths. Future agents prefer high-value paths, but evaporation prevents stale conventions from becoming permanent.

### Why interesting
A warehouse-specific routing system emerges from usage rather than static metadata.

---

## 51. Stigmergic SQL agents

### Inspiration
Social insects coordinate indirectly by modifying a shared environment.

### Transfer
Agents do not communicate through chat. They leave structured traces:

- join warning
- verified metric mapping
- failed hypothesis
- useful probe

Later agents coordinate via these traces.

### Research question
Can indirect coordination outperform explicit multi-agent conversation while using fewer tokens?

---

## 52. Novelty search instead of accuracy search

### Inspiration
Evolutionary novelty search rewards behavioral novelty rather than direct objective improvement.

### Transfer
When generating candidate SQLs, reward semantic diversity rather than likelihood. The verifier then searches a broader hypothesis space.

### Hypothesis
For difficult tasks, one low-probability structurally novel candidate may be more valuable than ten near-duplicates.

---

# XIII. Process Mining and Workflow Intelligence

## 53. Process mining over successful analyst behavior

### Inspiration
Process mining reconstructs workflow models from event logs.

### Transfer
Use historical analyst traces to learn how humans actually solve query classes:

- which metadata they inspect first
- when they run validation queries
- where they backtrack
- which artifacts they trust

Learn a workflow grammar rather than copying final SQL.

### Research angle
Imitate *problem-solving process*, not answer text.

---

## 54. Conformance checking for agent trajectories

### Inspiration
Process mining compares observed workflows against a normative process.

### Transfer
Define safety processes for high-risk analytics. An agent trajectory can be accurate yet non-conformant if it skipped mandatory checks.

### Example
Financial KPI query must verify reporting period and currency normalization.

### Output
Trajectory conformance certificate.

---

# XIV. Strange but Testable Meta-Ideas

## 55. Semantic dark matter benchmark

Study queries whose required business semantics are *not present anywhere* in schema/docs. Measure whether systems correctly identify underdetermination rather than hallucinate.

Contribution: benchmark epistemic limits, not generation power.

---

## 56. SQL hallucination conservation law

Hypothesis: adding more reasoning does not monotonically reduce error; it may transform obvious schema hallucinations into subtler semantic hallucinations.

Track how error mass moves between categories as compute increases.

---

## 57. Adversarially incomplete schemas

Instead of adversarial examples at the language level, systematically hide different classes of metadata and measure which semantic abilities degrade. This yields a causal map from information source to query capability.

---

## 58. Warehouse cartography benchmark

Give an agent an unseen warehouse and a fixed exploration budget *before* any user queries. The agent chooses what to inspect to build a semantic map. Later evaluate a stream of queries.

Research variable: what should an autonomous agent explore when it does not yet know future tasks?

---

## 59. Semantic debt

Like technical debt, every unsupported shortcut in a long-lived SQL agent accumulates future risk. Define semantic debt as unresolved assumptions stored in memory and measure their downstream maintenance cost.

A good system may prefer a slower first answer that prevents repeated future ambiguity.

---

## 60. Meta-scientist SQL agent

The agent maintains competing theories of the warehouse itself. Queries are not only tasks; they are experiments that update the warehouse theory. Over months, the system becomes a scientist of one organization’s data-generating process.

This is the furthest departure from one-shot Text2SQL:

`answer query` becomes a side effect of `learn the latent semantics of the data world`.

---

# XV. The strongest candidates from this batch

## Tier S: genuinely different and experimentally clean

1. **SLAM for Warehouses / Simultaneous Querying and Warehouse Mapping**
2. **Belief Revision for Long-Lived SQL Agents**
3. **SAT/SMT Semantic Completion + Minimal Conflict Cores**
4. **Semantic Error Epidemiology and R_semantic**
5. **Persistent Ambiguity / Persistent Homology of Candidate Semantics**
6. **Causal Intervention Consistency Benchmark**
7. **Renormalized Multi-Scale Schema Reasoning**
8. **Warehouse Cartography Pre-Exploration Benchmark**

## Tier A: unusual but plausible

- active-perception SQL
- legal burden-of-proof routing
- complementary learning systems for warehouse memory
- process-mined analyst workflows
- stigmergic multi-agent coordination
- real-options delayed semantic commitment
- sheaf-style local-to-global consistency
- optimal-transport schema linking

## Tier B: high-risk moonshots

- category-theoretic semantic compilation
- neural synchronization as confidence
- thermodynamic entropy-production metrics
- musical counterpoint agents
- blind-source-separation business semantics

---

# XVI. Suggested experiments that do not require a giant system

### Experiment A — persistent ambiguity
Generate candidates under increasing context and compute. Measure which disagreements disappear vs persist. Test whether persistence predicts human ambiguity labels.

### Experiment B — semantic SLAM
Stream 200 queries from one warehouse. Compare stateless agent vs episodic memory vs consolidated semantic map. Measure adaptation curve and memory-induced error propagation.

### Experiment C — SMT conflict cores
Construct ambiguous questions with incompatible constraints. Test whether an LLM+SMT system asks more targeted clarification questions than an LLM alone.

### Experiment D — semantic epidemiology
Plant one wrong high-centrality memory and one wrong low-centrality memory. Measure downstream corruption over a query stream. Estimate empirical `R_semantic`.

### Experiment E — multi-scale reasoning
Give the agent only domain/entity graph first, then selectively reveal tables/columns/values. Compare with full-schema prompting under 1000+ column conditions.

### Experiment F — causal intervention consistency
Create controlled intent interventions and test whether only the intended SQL components change. This requires no new natural-language gold SQL if transformations are generated from existing tasks.

---

# XVII. New umbrella thesis

A conventional Text2SQL system asks:

> “What SQL corresponds to this sentence?”

The ideas in this memo suggest a more radical research object:

> **A persistent epistemic agent that maintains, tests, revises, and maps beliefs about a data world.**

SQL is merely one action language through which that agent interrogates the world.

Under this view, the most important research variables become:

- belief state quality
- map quality
- contradiction handling
- exploration policy
- long-term semantic drift
- propagation of wrong beliefs
- local-to-global consistency
- interventional stability
- epistemic limits

That framing is sufficiently different from current single-query Text2SQL that it may support several independent research programs rather than one incremental architecture.