# Literature Corrections and Claim Hygiene

> **Date:** 2026-08-23  
> This file records corrections discovered during deeper literature checking. It exists to prevent an attractive but incorrect prior-art narrative from propagating into later proposals or papers.

## Correction 1 — SPARTA was mischaracterized in an earlier exploratory summary

An earlier round/chat characterization treated **SPARTA (ICLR 2026)** as if it were a Text2SQL query-repair method using why-not provenance to repair an end-user SQL query.

That characterization is **incorrect / materially misleading**.

The SPARTA work found during the Round-3 verification is a framework for constructing table-text question-answering benchmarks. Provenance/query refinement is used inside the **benchmark-generation pipeline** to create realistic, valid/non-empty queries. It should **not** be cited as the closest prior work for sparse result-feedback repair of LLM-generated Text2SQL.

### What should be cited instead for Result2SQL / DenoRepair

The relevant prior-art families are:

- Chapman & Jagadish, **Why Not?**, SIGMOD 2009 — why-not explanations for missing answers;
- ConQueR-style why-not query refinement — refined queries to recover missing answers;
- **A Framework for Query Refinement with User Feedback**, Journal of Systems and Software 2013 — direct expected/missing tuple feedback for query refinement;
- Buneman et al. — why/where provenance;
- Green et al. — provenance semirings;
- Meliou et al. — causality/responsibility for query answers/non-answers;
- PATSQL / Sickle / PBE — SQL synthesis from example outputs or demonstrations;
- **Speak to Your Parser**, ACL 2020 — Text2SQL correction from natural-language user feedback;
- modern SQL-debugging / repair agents and benchmarks.

### Consequence for novelty

The naive claim

```text
user marks missing/unexpected result tuples -> system repairs SQL
```

must be treated as **old prior art**, not a novelty claim.

The surviving research seam is narrower:

```text
sparse heterogeneous denotational constraints
+ complex executable-but-semantically-wrong analytical SQL
+ provenance/causality-guided structural localization
+ hidden-generalization / collateral-damage evaluation
```

## Correction policy

If later searches invalidate another claim, add it here rather than silently rewriting history. In this research program, a documented correction is evidence that the novelty process is functioning, not a failure to hide.