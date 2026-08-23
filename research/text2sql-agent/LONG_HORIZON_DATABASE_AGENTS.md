# Long-Horizon Database Agents: A Research Program Beyond One-Shot Text2SQL

> Central claim: the most interesting successor to Text2SQL may not be a better one-shot generator. It may be a persistent agent that learns one data environment over weeks or months, updates its beliefs, survives semantic drift, collaborates with humans, and improves the environment itself.

## 1. Why this is a different problem

Classic Text2SQL evaluates independent examples:

`question_i -> SQL_i`

A long-horizon database agent receives a stream:

`warehouse_0 + task_1 + task_2 + ... + task_T + environment changes`

Its state matters.

A decision on task 5 may improve or corrupt task 500.

This introduces variables absent from standard benchmarks:

- memory quality
- adaptation
- forgetting
- semantic drift
- cumulative cost
- trust calibration
- cross-task transfer
- error propagation
- map formation
- environment modification

## 2. Benchmark concept: DatabaseAgent-Stream

### Episode
One persistent warehouse and 100–1000 sequential tasks.

### Hidden events
At random points:

- schema rename
- new table
- deprecated metric
- changed business definition
- contradictory documentation
- ownership change
- stale memory injected

### Agent abilities

- inspect metadata
- execute read-only probes
- ask clarification
- store/revise memory
- create semantic notes
- abstain

Optional advanced track:

- propose dbt/metric-layer edits

### Evaluation

#### Task metrics
- semantic success
- cost
- latency

#### Stream metrics
- forward transfer
- backward interference
- adaptation lag after drift
- memory precision
- error reproduction number
- cumulative semantic debt
- number of repeated clarifications
- map coverage
- cumulative tool cost

## 3. Benchmark concept: Warehouse Cartography

Before any query arrives, give the agent a budget of K actions to explore an unknown warehouse.

The agent may inspect:

- tables
- columns
- relationships
- values
- docs
- lineage
- usage statistics

Then freeze exploration and evaluate future tasks.

### Research question
What should an agent learn about an environment when it does not know which questions will be asked later?

### Baselines
- random exploration
- centrality-based exploration
- schema coverage
- query-workload-trained policy
- curiosity / uncertainty policy

## 4. Benchmark concept: Semantic Drift Challenge

A long query stream uses a stable environment for the first phase. Then silently modify one concept.

Examples:

- `active_customer` changes definition
- `revenue` switches from gross to net
- canonical timestamp changes
- bridge table becomes mandatory

### Measure

- time until detection
- number of wrong answers before detection
- time to memory repair
- collateral forgetting

### Critical distinction
A model can adapt too fast and overwrite valid old semantics, or adapt too slowly and remain stale.

## 5. Benchmark concept: Memory Poisoning and Recovery

Give the agent a useful memory store, then inject one incorrect high-centrality memory.

Measure downstream impact.

### Example poison

`orders.customer_id is unique per row`

If believed, it may silently corrupt many future aggregation queries.

### Research questions

- can the agent detect inconsistent outcomes?
- can it quarantine the memory?
- can it identify dependent conclusions?
- can it recover the previous valid map?

## 6. Benchmark concept: Organizational Contradiction

Provide multiple authoritative-looking sources:

- schema comment
- dbt docs
- finance report
- dashboard query
- analyst note

Some disagree.

### Goal
The agent should not force a fake consensus.

Evaluate:

- source-priority reasoning
- conflict identification
- targeted clarification
- calibrated abstention
- audit trail quality

## 7. Benchmark concept: Semantic Infrastructure Improvement

Give repeated tasks that expose the same ambiguity.

An advanced agent may propose a durable asset:

- canonical metric
- glossary entry
- join rule
- dbt test
- ontology edge

Future tasks become easier if the proposal is correct.

### Objective
Minimize cumulative cost over the whole stream:

`answering cost + clarification cost + maintenance cost + semantic error cost`

### Strong research thesis
The optimal agent changes its environment to reduce future reasoning complexity.

## 8. Architecture idea: Database World Model

Persistent state could contain:

### Entity map
Known business entities and aliases.

### Relationship map
Join paths with confidence, cardinality, provenance, and last-verified time.

### Metric map
Definitions, owners, contexts, disagreements, and temporal versions.

### Incident memory
Past failures and repairs.

### Uncertainty map
Areas of the warehouse the agent knows it does not understand.

### Drift monitor
Evidence that beliefs may have become stale.

This is closer to a robot world model than a prompt memory.

## 9. Architecture idea: memory lifecycle

Every memory moves through states:

`hypothesis -> episodic support -> provisional rule -> validated rule -> stale -> contested -> retired`

Promotion depends on independent evidence.

Contradiction does not immediately delete a memory; it changes its state.

## 10. Architecture idea: semantic garbage collection

Long-lived agents accumulate obsolete beliefs.

Borrow garbage collection ideas:

- reference counting: how many current concepts depend on this memory?
- reachability: is this memory still used?
- generational collection: inspect new memories more frequently
- compaction: merge redundant semantic facts

### Research question
Can memory hygiene improve accuracy, not just storage efficiency?

## 11. Architecture idea: semantic version control

Represent warehouse knowledge as versioned state.

When a metric changes, preserve:

- prior definition
- new definition
- effective date
- migration evidence
- affected downstream concepts

### Benefit
Questions about historical periods may require historical semantics.

## 12. Architecture idea: branch-and-merge beliefs

Two teams may use different valid meanings of the same term.

Instead of forcing one global definition, maintain semantic branches:

- finance/revenue
- sales/revenue

Merge only where mappings are explicit.

### Research angle
Multi-tenant semantic memory without destructive global consensus.

## 13. Architecture idea: uncertainty-aware map expansion

The agent maintains a frontier between known and unknown warehouse regions.

When repeated tasks hit the frontier, it spends exploration budget to expand the map.

### Analogy
Autonomous exploration in robotics.

## 14. Architecture idea: incident-driven learning

Only expensive/important failures should trigger durable memory updates.

Routine successful tasks may not deserve storage.

### Goal
Avoid converting accidental correlations into permanent rules.

## 15. New metrics

### Semantic half-life
How long a memory remains useful before becoming stale.

### Adaptation lag
Number of tasks between environment change and reliable recovery.

### Corruption radius
How many future decisions one wrong memory affects.

### Semantic debt
Accumulated unresolved assumptions that increase future cost.

### Map efficiency
Task success per unit of persistent warehouse knowledge.

### Clarification amortization
How many future clarification questions are avoided by one durable semantic update.

### Trust recovery time
How quickly the system returns to calibrated behavior after detecting a bad belief.

## 16. New learning setups

### Offline consolidation
Replay historical trajectories while idle and promote only repeatedly supported semantic patterns.

### Continual distillation
Distill stable warehouse-specific knowledge into a small local model or retrieval policy.

### Active replay
Prioritize replay of memories with high centrality or high uncertainty.

### Counterfactual replay
Re-run old tasks under new semantic definitions to identify which memories/results are invalidated.

## 17. Multi-user angle

Different users may teach conflicting semantics.

The agent needs:

- identity/context-aware memory
- authority weighting
- team-local concepts
- global concepts
- provenance

### Benchmark idea
Simulate users from finance, product, and sales asking semantically overlapping but non-identical questions.

## 18. Multi-agent angle

Several agents share one warehouse map.

Questions:

- how should discoveries propagate?
- how are conflicting updates resolved?
- can one bad agent poison shared memory?
- should agents use stigmergic marks instead of direct chat?

### Distributed systems analogy
The warehouse knowledge base becomes a replicated semantic state machine.

## 19. Research programs

### Program A — Continual Semantic Mapping
Focus: learning a warehouse map from task streams.

### Program B — Memory Integrity
Focus: poisoning, contradiction, revision, quarantine, provenance.

### Program C — Drift Adaptation
Focus: temporal versions of organizational semantics.

### Program D — Environment Improvement
Focus: agents creating durable semantic assets.

### Program E — Shared Organizational Intelligence
Focus: multi-user/multi-agent semantic state.

## 20. Why this could matter more than another Spider score

One-shot benchmark accuracy does not answer the industrial question:

> If I deploy this agent on my warehouse for six months, will it become safer and cheaper—or will it accumulate invisible semantic corruption?

That question is largely unmeasured.

A strong long-horizon benchmark could open a research direction spanning:

- Text2SQL
- agent memory
- continual learning
- data governance
- interactive learning
- semantic layers
- multi-agent systems

## 21. Minimal pilot

No giant infrastructure is required initially.

Use one medium database and generate a 200-task stream.

At task 100, change one important semantic rule.

Compare:

1. stateless agent
2. append-only retrieval memory
3. versioned memory
4. belief-revision memory

Measure:

- accuracy before drift
- wrong answers after drift
- detection delay
- recovery delay
- cost

If versioned/revisable memory does not materially help, kill the long-horizon thesis early.

## 22. Long-term vision

The endpoint is not a model that can translate arbitrary questions into arbitrary SQL.

It is a **resident data-world agent** that gradually becomes competent in one organization:

- it knows what the organization means
- knows what it does not know
- notices when meanings change
- records why it believes things
- does not let one bad memory silently contaminate the future
- creates durable semantic infrastructure
- asks fewer, better questions over time

That is a qualitatively different research target from classic Text2SQL.