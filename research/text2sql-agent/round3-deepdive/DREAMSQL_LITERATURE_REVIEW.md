# Round 3 Deep Dive — DreamSQL / Semantic Library Learning

> **Date:** 2026-08-23  
> **Workflow role:** `research-lit` → candidate-specific deep literature review  
> **Status:** grounded research memo; not a novelty verdict by itself.  
> **Verification note:** live web search was used to verify titles/venues/arXiv identifiers where available. The repository's `verify_papers.py` helper is not invokable from this connector session, so under the ARIS citation contract these entries should still be treated as **[UNVERIFIED-HELPER]** until helper-level verification is run in a local ARIS environment.

## 1. Question

Can a long-lived Text2SQL agent improve not by retrieving old SQL, but by **inventing a compact, compositional, warehouse-specific semantic language** from many verified solutions?

Project codename: **DreamSQL**. Academic name proposed later: **SemLibSQL**.

The important distinction is:

```text
query memory:    old programs -> retrieve/reuse old programs
library learning: old programs -> induce new abstractions -> solve new compositions
```

If the method is only "store useful SQL snippets as skills," it is not sufficiently novel. The surviving research question is whether **semantics-aware program-library induction** can discover reusable business operators that are hidden across syntactically diverse SQL programs and improve held-out compositional generalization.

---

## 2. Closest program-synthesis literature

### 2.1 DreamCoder — the core inspiration

**[UNVERIFIED-HELPER] Ellis et al., _DreamCoder: Growing Generalizable, Interpretable Knowledge with Wake-Sleep Bayesian Program Learning_, PLDI 2021 / arXiv:2006.08381.**

DreamCoder jointly learns:

- a domain-specific program library;
- a neural search policy;
- reusable abstractions discovered by refactoring solved programs.

The key lesson for Text2SQL is **not replay**. It is that a learner can change its own representational language as experience accumulates.

Directly applying DreamCoder to SQL, however, would be weak novelty: SQL already has compositional syntax, and syntactic common-subexpression mining may simply rediscover CTE-shaped fragments rather than business semantics.

### 2.2 Stitch — scalable library learning

**[UNVERIFIED-HELPER] Bowers et al., _Top-Down Synthesis for Library Learning_, POPL 2023, DOI:10.1145/3571234.**

Stitch makes corpus-guided abstraction learning substantially faster than DreamCoder-style compression. This is a serious baseline and implementation primitive for any proposed library learner.

A Text2SQL paper therefore cannot claim novelty from "mine frequent subprograms." It needs a SQL-/semantics-specific reason why ordinary syntactic library learning is insufficient.

### 2.3 LILO — library learning plus LLM-guided synthesis/documentation

**[UNVERIFIED-HELPER] Grand et al., _Learning Interpretable Libraries by Compressing and Documenting Code_, ICLR 2024.**

LILO combines LLM-guided program synthesis, Stitch-style library learning, and automatic documentation. It substantially narrows the novelty space for a method that simply asks an LLM to name or document mined abstractions.

Therefore, in SemLibSQL, **LLM naming/docstrings should be optional presentation machinery, not the contribution**.

### 2.4 Earlier library-learning work

Classic work on Bayesian program learning and EC-style library induction already establishes the principle that repeated solutions can be compressed into reusable abstractions. The Text2SQL contribution has to come from the specific difficulty of discovering **semantic equivalence and business meaning across SQL programs**, plus the downstream Text2SQL evaluation.

---

## 3. Closest agent-skill literature

The generic-agent literature is rapidly converging on persistent skill libraries, which makes "agents learn reusable skills" a saturated claim.

### 3.1 Voyager

**[UNVERIFIED-HELPER] Wang et al., _Voyager: An Open-Ended Embodied Agent with Large Language Models_, arXiv:2305.16291.**

Voyager stores executable code skills and composes/retrieves them for later tasks. This already demonstrates the broad pattern `experience -> executable skill library -> reuse`.

### 3.2 SAGE and recent self-improving skill libraries

**[UNVERIFIED-HELPER] _Reinforcement Learning for Self-Improving Agent with Skill Library_ (SAGE), arXiv:2512.17102 / ACL 2026.**

Recent work such as SAGE, SkillFoundry, EvoSkills, SkillOps, HASP, and SkillCraft explores learning, validating, maintaining, composing, or benchmarking agent skills. Collectively they make generic claims about autonomous skill acquisition weak.

The domain-specific opening is narrower:

> SQL solutions have multiple surface forms, dialects, optimizer rewrites, CTE decompositions, and equivalent relational plans. A useful learned primitive must capture a **stable semantic/business operation**, not a token sequence or tool macro.

---

## 4. Closest Text2SQL memory literature

### 4.1 AgentSM

**[UNVERIFIED-HELPER] Biswal et al., _AgentSM: Semantic Memory for Agentic Text-to-SQL_, arXiv:2601.15709.**

AgentSM stores prior execution traces as interpretable structured programs and retrieves/reuses them to shorten trajectories and improve Text2SQL performance.

This is the most important Text2SQL baseline. A new method must prove it does something stronger than structured trace retrieval.

### 4.2 Crystallization / reusable verified query memory

**[UNVERIFIED-HELPER] Wang et al., _From Test-Time Scaling to Reusable Memory: Measuring Crystallization in Text-to-SQL_, arXiv:2608.07213.**

This very recent work explicitly studies turning corrected/verified test-time solutions into reusable memory and reports held-out same-database transfer. It also finds that database-specific content is a major source of value.

This sharply raises the bar for DreamSQL. "Verified history helps later queries" is already a result. The question must instead be:

> Does inducing **new reusable operators** from many histories beat retaining those histories themselves, especially on compositions not present in memory?

### 4.3 Continual learning from human feedback

**[UNVERIFIED-HELPER] Cook et al., _Continual Learning of Domain Knowledge from Human Feedback in Text-to-SQL_, arXiv:2511.10674.**

This line distills human corrections and domain knowledge into persistent memory. Again, persistence is not novel. Abstraction induction and compositional reuse must be the focus.

### 4.4 GATE and semantic grounding memory

**[UNVERIFIED-HELPER] _Bootstrapping Semantic Layer from Execution for Text-to-SQL_ (GATE), arXiv:2606.05634.**

GATE uses execution to test grounding hypotheses and stores validated semantic groundings. This is close to "learn business semantics over time," but its learned units are grounding facts/hypotheses, not necessarily newly induced higher-order program operators composed across multiple verified solutions.

---

## 5. Why ordinary syntactic library learning is not enough for SQL

A warehouse-specific semantic abstraction can appear in many syntactic shapes.

Example hidden motif: **latest valid snapshot per customer as of date D**.

Equivalent implementations may use:

- `ROW_NUMBER() OVER (...) QUALIFY = 1`;
- a max-date subquery joined back;
- `DISTINCT ON` in Postgres;
- nested CTEs;
- a pre-existing dbt model;
- a lateral join;
- an engine-specific aggregate function.

A token-/AST-based compressor may see unrelated fragments. Yet all instantiate one semantic operator:

```text
latest_valid_snapshot(
    entity = Customer,
    key = customer_id,
    event_time = effective_at,
    as_of = D
)
```

Therefore the plausible algorithmic contribution is **semantic canonicalization before library induction**.

Possible canonicalization signals:

1. typed relational IR;
2. conservative e-graph rewrite classes;
3. schema/entity/grain annotations;
4. execution equivalence on generated discriminative databases;
5. lineage/provenance signatures;
6. invariants such as output grain, key preservation, monotonicity, and duplicate sensitivity.

Only then should recurring structures be mined into a library.

---

## 6. Proposed surviving method: SemLibSQL

### 6.1 Dominant contribution

> **Given a stream/corpus of verified Text2SQL solutions from one warehouse, automatically induce a compact typed library of reusable semantic operators from behaviorally equivalent but syntactically diverse SQL programs, and use the learned library to solve held-out compositions more accurately and cheaply than episodic query memory.**

### 6.2 Pipeline

```text
verified NL/SQL experiences
        |
        v
SQL parse + schema grounding
        |
        v
typed semantic plan
        |
        v
semantic canonicalization
(e-graph / safe rewrites / behavior tests)
        |
        v
corpus-level abstraction mining
        |
        v
candidate semantic operators
        |
        v
execution/evidence validation
        |
        v
warehouse-specific semantic library
        |
        +----> future NL -> operator composition -> SQL lowering
```

### 6.3 Example learned operators

- `latest_snapshot(entity, as_of)`
- `active_customer(as_of)`
- `recognized_revenue(period, currency)`
- `dedup_events(key, event_id)`
- `fiscal_period(date, calendar)`
- `bridge_join(left_entity, right_entity, bridge)`
- `eligible_population(metric_context)`
- `scd2_state(entity, as_of)`

The library should be **parameterized**. Memorizing a fixed CTE is not enough.

---

## 7. Core novelty claims to test

### Claim C1 — Semantic abstraction, not retrieval

The system induces reusable operators that are not present verbatim as complete stored programs.

**Closest prior:** AgentSM, Crystallization, continual memory.  
**Novelty risk:** HIGH if evaluation permits nearest-neighbor leakage.  
**Required proof:** held-out composition split with no near-duplicate full SQL.

### Claim C2 — Semantics-aware canonicalization is necessary

Mining after semantic canonicalization discovers higher-purity reusable operators than syntactic-only Stitch-style compression.

**Closest prior:** DreamCoder/Stitch/LILO.  
**Required proof:** ablation `syntactic mining` vs `semantic canonicalization + mining`.

### Claim C3 — Learned operators improve compositional generalization

If training experience contains motifs A, B, C and some combinations but withholds A+C or A+B+C, learned primitives improve on those unseen compositions.

This is the strongest scientifically interesting claim.

### Claim C4 — Library growth reduces long-horizon reasoning cost without increasing silent error

As experience grows, token/tool cost per future query should fall while semantic accuracy is stable or improves.

---

## 8. Cheap falsification pilot

Do not train a new LLM initially.

### Corpus

- 3 databases / projects;
- 8–12 recurring semantic motifs;
- 200–500 verified SQL solutions total;
- multiple syntactic realizations per motif;
- at least two SQL dialects if feasible.

### Split

For motifs A/B/C:

```text
train: A, B, C, A+B, B+C
held-out: A+C, A+B+C
```

Enforce no near-duplicate SQL retrieval across split.

### Baselines

1. strong stateless Text2SQL;
2. nearest verified SQL retrieval;
3. structured trace memory (AgentSM-like);
4. full verified-query memory (Crystallization-like);
5. manual/oracle primitive library;
6. syntactic Stitch-style library learner;
7. **SemLibSQL**.

### Metrics

- semantic/execution accuracy;
- held-out composition accuracy;
- first-attempt success;
- tokens/tool calls;
- trajectory length;
- library compression ratio;
- primitive reuse rate;
- abstraction purity;
- negative-transfer rate;
- cross-dialect transfer where applicable.

### Kill conditions

Kill or radically reframe if any of the following occurs:

- <2–3 percentage-point gain over strong retrieval/memory on held-out composition, with uncertainty overlapping heavily;
- gains vanish under matched context/token budget;
- learned abstractions are mostly SQL syntax fragments rather than stable semantic motifs;
- negative transfer caused by learned operators cancels their benefit;
- a manually authored primitive library dominates, showing automatic induction adds little.

---

## 9. Strongest reviewer attacks

### Attack 1: "This is DreamCoder/Stitch applied to SQL."

Valid unless the paper demonstrates that **SQL semantic equivalence/canonicalization changes what abstractions are discoverable**, and that these abstractions improve a Text2SQL-specific compositional split.

### Attack 2: "Agent skills already learn reusable procedures."

Valid unless the learned objects are shown to be database-semantic operators rather than generic procedural macros.

### Attack 3: "The benchmark is synthetic and toy."

A motif-controlled benchmark is useful for causal isolation, but the paper also needs at least one enterprise-like/open real workload (e.g. Spider 2.0/dbt-style repository or open SQL workload) with manually audited semantic motifs.

### Attack 4: "The model is just retrieving operator names instead of learning semantics."

Need intervention tests and parameter substitutions showing operators behave correctly under new entities, time windows, dimensions, and dialect lowerings.

### Attack 5: "The library leaks the test set."

Need temporal/workload or compositional split, deduplication, and explicit nearest-neighbor analysis.

---

## 10. Literature-grounded verdict

**Verdict: PROCEED WITH CAUTION.**

- Generic `SQL agent + learned skill library`: **LOW novelty**.
- `DreamCoder/Stitch applied directly to SQL AST`: **LOW–MEDIUM novelty**.
- **Semantics-aware canonicalization + automatic typed semantic-operator induction + held-out compositional Text2SQL evaluation**: **MEDIUM–HIGH novelty** and experimentally falsifiable.

A reasonable current novelty estimate is **~7/10**, contingent on the semantic-canonicalization mechanism being substantive and the evaluation proving composition rather than replay.

The dominant paper should be kept this narrow. Result correction and automatic semantic-view creation should not be bundled into the first method.