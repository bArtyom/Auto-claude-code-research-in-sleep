# Round 3 Deep Dive — Result2SQL / Denotational SQL Repair

> **Date:** 2026-08-23  
> **Workflow role:** `research-lit` → candidate-specific deep literature review  
> **Status:** grounded research memo.  
> **Verification note:** titles/metadata were checked through live web search, but `verify_papers.py` is unavailable in this connector session; entries remain **[UNVERIFIED-HELPER]** under the ARIS citation contract until local helper verification.

## 1. Starting hypothesis

The attractive product interaction is simple:

```text
question -> SQL -> result table
                    |
                    +-- user: this row should not be here
                    +-- user: this row is missing
                    +-- user: this cell is wrong
                    +-- user: these two rows are the same entity
                    +-- user: the total should be around X
                              |
                              v
                         SQL repair
```

The initial intuition was that sparse corrections on the **denotation** could be a novel alternative to asking non-expert users to explain SQL bugs in natural language.

The literature check substantially narrows this claim: expected/missing tuple feedback for query refinement is old database work. Therefore the paper cannot claim "first result-feedback SQL repair." The surviving opportunity is **provenance-guided structural repair of complex LLM-generated SQL under sparse, heterogeneous output constraints, evaluated for generalization and collateral damage**.

---

## 2. Foundational provenance and causality

### 2.1 Why/where provenance

**[UNVERIFIED-HELPER] Buneman, Khanna, Tan, _Why and Where: A Characterization of Data Provenance_, ICDT 2001.**

Database provenance formalizes how output tuples and values depend on source tuples. This is the natural localization substrate for output-level feedback.

### 2.2 Provenance semirings

**[UNVERIFIED-HELPER] Green, Karvounarakis, Tannen, _Provenance Semirings_, PODS 2007.**

This gives an algebraic view of provenance propagation through relational queries and reinforces that result corrections can be traced to structured query/data causes rather than handled purely through language-model reflection.

### 2.3 Query causality and responsibility

**[UNVERIFIED-HELPER] Meliou et al., work on causality/responsibility for query answers and non-answers, PVLDB 2010 and follow-ups.**

Responsibility ranks which tuples are most causally responsible for an answer. For LLM-generated SQL repair, a related idea is to rank **query operators / predicates / joins** by their responsibility for a flagged result error.

---

## 3. Why-not explanations and query refinement

This is the closest prior-art family and it invalidates the naive novelty claim.

### 3.1 Why-not provenance

**[UNVERIFIED-HELPER] Chapman & Jagadish, _Why Not?_, SIGMOD 2009.**

Why-not explanations study why expected tuples are missing and identify the query manipulation responsible for their disappearance.

### 3.2 ConQueR and refined queries

Follow-up systems automatically generate refined queries that include missing answers. This already establishes a route from missing-result feedback to query modification.

### 3.3 Query refinement with explicit user feedback — direct collision

**[UNVERIFIED-HELPER] _A Framework for Query Refinement with User Feedback_, Journal of Systems and Software, 2013, DOI:10.1016/j.jss.2013.01.069.**

This is a direct collision with a naive Result2SQL formulation: users identify false positives (unexpected tuples) and false negatives (expected missing tuples), and the system refines the query accordingly.

**Consequence:**

> `unexpected/missing tuples -> refine query` is **not novel**.

Any Result2SQL paper must explicitly cite this line and move to a harder setting and a different scientific objective.

---

## 4. Example-guided SQL synthesis

### 4.1 PATSQL

**[UNVERIFIED-HELPER] _PATSQL: Efficient Synthesis of SQL Queries from Example Tables_, arXiv:2010.05807.**

Input/output examples can synthesize SQL directly. This reduces the novelty of "infer SQL from corrected output examples."

### 4.2 Sickle

**[UNVERIFIED-HELPER] _Synthesizing Analytical SQL Queries from Computation Demonstration_ (Sickle), arXiv:2204.07102.**

Sickle supports analytical SQL from demonstrations, showing that complex SQL can be constrained by observed computation/examples.

### 4.3 Programming by Example broadly

PBE literature has long treated examples as specifications. Therefore the novelty cannot be the supervision modality alone; it must be the **repair setting, localization mechanism, sparse/partial constraints, and generalization metric**.

---

## 5. Interactive NL2SQL repair and explanations

### 5.1 Speak to Your Parser

**[UNVERIFIED-HELPER] _Speak to Your Parser: Interactive Text-to-SQL with Natural Language Feedback_, ACL 2020.**

Incorrect SQL plus free-form natural-language feedback is used to produce corrections. This is a strong baseline for any human-feedback repair system.

### 5.2 DIY and non-expert debugging interfaces

Interactive systems have already exposed mappings, explanations, and subdatabases to help non-experts identify or repair NL2SQL errors.

### 5.3 Past feedback reuse

Recent Text2SQL work also reuses prior user corrections/history. Thus Result2SQL should not be mixed with a generic memory claim.

---

## 6. SQL debugging and modern LLM repair

Recent SQL-debugging benchmarks and agents make "LLM repairs erroneous SQL" crowded. DBMS-guided systems can identify syntax/schema/execution failures and iteratively patch candidate SQL.

The useful gap is **semantically wrong but executable SQL**, where the only human signal is a sparse result constraint.

Examples:

- wrong join path but plausible rows;
- missing `DISTINCT` causing inflated counts;
- wrong entity grain;
- wrong time anchor;
- incorrect numerator/denominator;
- wrong SCD snapshot semantics;
- predicate in the wrong stage (`WHERE` vs post-aggregation logic);
- null/outer-join semantics;
- window-frame error.

These are difficult for ordinary DBMS feedback because execution succeeds.

---

## 7. Revised research object: DenoRepair

Academic working name: **DenoRepair — Provenance-Guided Structural Repair from Sparse Denotational Constraints**.

### 7.1 Input

- natural-language question;
- executable but semantically wrong SQL;
- database/schema/context;
- **1–3 sparse output-level constraints**, e.g.:
  - `must_include(tuple)`;
  - `must_exclude(tuple)`;
  - `cell(row, col) = value`;
  - `same_entity(row_i, row_j)`;
  - `aggregate in [L,U]`;
  - `rank(A) > rank(B)`;
  - `count_distinct(entity) = k` for a small inspected subset;
  - a user-edited result slice.

### 7.2 Output

A repaired SQL program that satisfies the feedback while preserving unrelated correct semantics.

### 7.3 Method sketch

```text
wrong executable SQL + sparse result constraint
                 |
                 v
          provenance / influence
                 |
                 v
       candidate responsible regions
   (join / predicate / group / window / grain)
                 |
                 v
       typed structural edit proposals
                 |
                 v
     execute + check feedback constraints
                 |
                 v
      hidden counterexample / regression tests
                 |
                 v
           minimal safe repair
```

The LLM proposes structural edits, but provenance/causality should **shrink and rank the edit space**.

---

## 8. The key scientific concept: collateral damage

A repair can trivially overfit the observed correction.

Example: user says customer `B` should not appear. A bad repair adds:

```sql
WHERE customer_id <> 'B'
```

It satisfies the visible feedback but does not fix the underlying semantic bug.

Therefore evaluation must include **hidden semantic correctness** and **collateral damage**.

Define for a repaired program `q'` relative to original `q` and target `q*`:

```text
visible_feedback_satisfaction(q')
hidden_semantic_accuracy(q', q*)
collateral_damage = correct_behavior_of_q_changed_by_q'
structural_edit_cost(q -> q')
```

A useful repair should satisfy the feedback **and** generalize to unseen rows/databases/perturbations.

This is a stronger objective than classical tuple-specific refinement.

---

## 9. Core novelty claims

### C1 — Heterogeneous sparse denotational constraints for complex SQL

Not just expected/missing tuples; support multiple output constraint types on joins, aggregation, windows, grain, and temporal semantics.

**Novelty:** MEDIUM.  
**Risk:** PBE and query-refinement literature can still be cited as close ancestors.

### C2 — Provenance-guided localization reduces repair search

Use provenance/causality/influence to identify responsible AST/operator regions before LLM repair.

**Novelty:** MEDIUM if implemented specifically for modern analytical SQL.  
**Risk:** explanation/query-debugging literature.

### C3 — Repairs generalize beyond visible feedback

Measure full denotation correctness on hidden data/perturbations and collateral damage.

**Novelty:** potentially HIGHER as an evaluation formulation if prior query-refinement systems mostly optimize observed feedback satisfaction.

### C4 — Non-expert result editing is a competitive feedback interface

Compare result edits with NL bug descriptions under equal interaction budget.

This is partly HCI/product research; do not overclaim algorithmic novelty from it.

---

## 10. Experimental design

### 10.1 Build semantically wrong but executable candidates

Start from correct queries and inject typed semantic mutations:

- join type/path;
- join key;
- aggregation function;
- `DISTINCT` removal/addition;
- group grain;
- date field / boundary;
- numerator/denominator;
- null semantics;
- window partition/order/frame;
- SCD snapshot logic;
- filter placement;
- bridge-table omission.

Keep only mutants that execute and yield plausible but incorrect results.

### 10.2 Generate sparse feedback

For each mutant, expose only 1–3 constraints sampled from differences to the gold denotation. Do not expose the gold SQL.

### 10.3 Baselines

1. full SQL regeneration from question/context;
2. LLM self-repair using execution result only;
3. natural-language feedback baseline;
4. classical query refinement for predicate-like cases where applicable;
5. PBE/example-guided synthesis baseline where tractable;
6. LLM repair with sparse denotational feedback but **no provenance localization**;
7. **DenoRepair**.

### 10.4 Metrics

- full semantic/execution correctness after repair;
- feedback satisfaction;
- collateral damage;
- edit distance / structural edit count;
- interactions required;
- tokens/tool calls;
- localization accuracy: did the system identify the true mutated operator family?;
- generalization to a perturbed database instance.

---

## 11. Cheap falsification pilot

Use ~150–300 mutated queries across 3–5 databases, with 6 high-value semantic mutation families.

No model training is initially required.

Compare:

```text
LLM + sparse result feedback
vs
LLM + provenance-ranked edit region + sparse result feedback
```

**Kill threshold:** if provenance-guided localization fails to improve full repair success, collateral damage, or interaction efficiency at matched model/tool budget, the main method is not justified.

---

## 12. Strongest reviewer attacks

### Attack 1: "This was done in query refinement in 2013."

Correct against the naive claim. The paper must clearly state that expected/missing tuple refinement is prior art and position the contribution around complex LLM-generated analytical SQL, heterogeneous partial constraints, structural localization, and hidden-generalization evaluation.

### Attack 2: "This is just PBE."

Need to show that the system starts from a mostly-correct candidate and repairs a localized semantic defect with extremely sparse feedback, rather than synthesizing from a complete example table.

### Attack 3: "Provenance only tells you data lineage, not the wrong SQL operator."

This is a real technical risk. The method needs an explicit mapping from feedback/provenance signatures to candidate AST regions and must measure localization quality.

### Attack 4: "The synthetic mutants make localization unrealistically easy."

Need real LLM error logs or naturally occurring incorrect candidates in addition to controlled mutation tests.

### Attack 5: "A stronger frontier model can simply regenerate correctly."

Matched-compute baselines are mandatory. The paper must demonstrate value particularly when the original query is long/mostly correct and full regeneration risks regressions.

---

## 13. Literature-grounded verdict

**Naive Result2SQL: ABANDON as a novelty claim.**

`expected/missing result tuples -> query refinement` has direct prior work.

**DenoRepair formulation: PROCEED WITH CAUTION.**

Estimated novelty: **~6/10**. The strongest paper angle is likely a **new repair benchmark/formulation plus provenance-guided structural method**, not a claim that result-level feedback itself is new.

This should be treated as a separate second research line from SemLibSQL, not bundled into the first paper.