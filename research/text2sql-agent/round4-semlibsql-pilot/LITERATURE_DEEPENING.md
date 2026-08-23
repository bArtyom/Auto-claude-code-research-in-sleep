# Round 4 — SemLibSQL Literature Deepening

> Date: 2026-08-23
> Workflow role: `research-lit`
> Focus: program-library learning, semantic canonicalization, SQL equivalence, workload mining, and compositional Text2SQL.

## 0. Updated research question

The narrow question is no longer whether prior SQL helps future SQL. That is established by memory/retrieval work. The question is:

> Can verified SQL histories be transformed into a **new warehouse-specific semantic language** whose induced operators generalize compositionally beyond episodic query reuse?

This round specifically searches for prior work that could invalidate that claim.

---

## 1. Program-library learning is a strong neighboring field

### DreamCoder

DreamCoder (Ellis et al., PLDI 2021) jointly learns a reusable program library and search policy from solved synthesis problems. Its wake-sleep procedure repeatedly compresses recurring program structure into new abstractions.

Implication: SemLibSQL cannot claim novelty for "automatically invent reusable functions from solved programs." That mechanism is mature.

### Stitch

Stitch / Top-Down Synthesis for Library Learning (Bowers et al.) performs corpus-guided library learning dramatically faster than DreamCoder-style bootstrapped search. It uses shared syntactic structure to propose abstractions and is a mandatory baseline for any corpus-compression claim.

Implication: a SQL-specific proposal must beat or complement syntactic library learning; otherwise it is just an application of Stitch.

### babble / anti-unification + e-graphs

Work on anti-unification and e-graph-based library learning shows that abstractions can be mined over equivalence classes rather than raw syntax. This substantially narrows the novelty of "use e-graphs before abstraction mining."

Implication: SemLibSQL should not claim e-graphs as the contribution. The contribution must be the **SQL/warehouse semantic equivalence relation and typed operator interface**.

### ReGAL

ReGAL: Refactoring Programs to Discover Generalizable Abstractions (ICML 2024) is especially close. It learns reusable functions from a small corpus of existing programs, verifies/refines abstractions by execution, and shows downstream synthesis gains.

This is the most important new collision found in Round 4.

**Novelty consequence:**

A generic claim of

`program corpus -> reusable functions -> better generation`

is already covered.

SemLibSQL survives only if it proves that its SQL-specific semantic canonicalization creates abstractions that **general program-refactoring/library learners cannot recover from surface program structure alone**, and that those abstractions improve held-out warehouse composition.

---

## 2. SQL workload mining is not library learning, but it is a relevant baseline family

Prior database work mines SQL workloads to identify recurring query/exploration patterns. Examples include work on mining SQL workloads for analysis behavior and industrial workload-pattern mining such as Alibaba Workload Miner.

These methods typically cluster queries or exploration trajectories for diagnosis, recommendation, forecasting, or optimization. They do not generally promote patterns into typed semantic operators used by a Text2SQL generator.

Nevertheless, they establish that:

- recurring structure in SQL workloads is not a new observation;
- SQL query templates/patterns are a standard database abstraction;
- any SemLibSQL evaluation should compare against simple workload-template induction.

Therefore include a **template miner** baseline, not just token and AST clustering.

---

## 3. SQL semantic equivalence already has strong formal tools

### Cosette

Cosette is an automated SQL equivalence prover. It can return a proof of equivalence or a counterexample database for supported SQL fragments.

### SQLSolver

SQLSolver (SIGMOD 2024) proves SQL query equivalence using linear integer arithmetic and SMT solving, supporting broad practical query classes.

### Equality saturation and database optimization

Recent work explicitly connects equality saturation with database query optimization. SQL optimizers already reason over large spaces of equivalent relational expressions.

**Novelty consequence:**

SemLibSQL cannot claim to introduce semantic SQL equivalence or equality saturation to databases. Instead, formal equivalence tools should be treated as a **high-precision canonicalization oracle for the supported fragment**.

A useful system design is therefore hybrid:

1. cheap structural normalization;
2. formal equivalence when supported;
3. counterexample search / execution signatures when formal proof is unavailable;
4. `UNKNOWN` rather than unsafe merge.

This conservative partial-equivalence stance is stronger scientifically than claiming complete SQL canonicalization.

---

## 4. Compositional generalization already has Text2SQL benchmarks

Spider-CG and later work on compositional generalization in Text2SQL show that models struggle when known semantic components appear in unseen combinations. Context-dependent variants such as CoSQL-CG / SParC-CG similarly recombine modification patterns.

These benchmarks are important methodological precedents, but their unit of composition is generally dataset-derived language/SQL components rather than **warehouse-specific induced semantic operators learned from a history stream**.

Therefore SemLibSQL should not claim "first compositional Text2SQL benchmark." Instead:

> We introduce a workload-history split designed specifically to distinguish **episodic memory** from **learned reusable semantic abstractions**.

The split must ensure that primitives are individually evidenced in history while complete target compositions are absent.

---

## 5. Recent Text2SQL work tightens the memory baseline

### AgentSM (2026)

AgentSM stores interpretable structured programs derived from prior execution traces and reuses them in future Text2SQL trajectories.

### GATE (2026)

GATE bootstraps missing semantic grounding from execution and accumulates supported grounding memories.

### FINER-SQL (2026)

FINER-SQL explicitly uses verified traces as a memory-alignment reward during RL.

**Consequence:**

A weak nearest-neighbor SQL retrieval baseline is no longer enough. The main comparison must be against a **structured verified-memory baseline** that can reuse partial prior solution structure.

---

## 6. Updated novelty boundary

The strongest defensible novelty statement is now:

> **SemLibSQL learns a typed warehouse-specific semantic operator library from verified SQL experience by canonicalizing SQL across syntactically different but semantically equivalent realizations, and evaluates whether that induced language enables unseen motif composition beyond structured verified-query memory.**

The proposal is *not* novel merely because it uses:

- program-library learning;
- e-graphs;
- anti-unification;
- execution-based abstraction validation;
- SQL equivalence checking;
- workload mining;
- compositional splits;
- semantic memory.

The scientific delta is the combination of:

1. **warehouse-semantic canonicalization**;
2. **typed abstraction induction over those semantic classes**;
3. **strict memory-vs-language evaluation on held-out compositions**.

---

## 7. Mandatory baselines after this literature round

1. lexical/token query clustering;
2. normalized SQL-template miner;
3. AST anti-unification;
4. Stitch-like syntactic library learning;
5. ReGAL-like execution-validated refactoring library;
6. structured verified semantic memory (AgentSM-style);
7. formal-equivalence-assisted semantic canonicalization;
8. human/oracle semantic library.

Without ReGAL/Stitch and structured memory baselines, the paper would be under-controlled.

---

## 8. References to verify/cite later

- Ellis et al. DreamCoder: Bootstrapping Inductive Program Synthesis with Wake-Sleep Library Learning. PLDI 2021.
- Bowers et al. Top-Down Synthesis for Library Learning / Stitch.
- Cao et al. babble: Learning Better Abstractions with E-Graphs and Anti-Unification.
- Stengel-Eskin, Prasad, Bansal. ReGAL: Refactoring Programs to Discover Generalizable Abstractions. ICML 2024.
- Chu et al. Cosette: An Automated Prover for SQL.
- Ding et al. Proving Query Equivalence Using Linear Integer Arithmetic / SQLSolver. SIGMOD 2024.
- Liu et al. Exploring the Compositional Generalization in Context Dependent Text-to-SQL Parsing. 2023.
- Gan et al. Measuring and Improving Compositional Generalization in Text-to-SQL via Component Alignment. 2022.
- Biswal et al. AgentSM: Semantic Memory for Agentic Text-to-SQL. 2026.
- Lee et al. Bootstrapping Semantic Layer from Execution for Text-to-SQL / GATE. 2026.
- Hoang et al. FINER-SQL. 2026.

## 9. Round-4 literature verdict

**SemLibSQL remains viable, but the novelty margin is narrower than Round 3.**

The key threat is ReGAL plus general library learning. The project should now be killed if SQL-specific semantic canonicalization fails to deliver abstractions that syntactic/execution-refactoring baselines cannot match.