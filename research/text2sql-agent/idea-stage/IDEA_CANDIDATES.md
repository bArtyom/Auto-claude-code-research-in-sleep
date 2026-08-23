# Text2SQL × Agent Idea Candidates — Final Shortlist

**Date:** 2026-08-23  
**Status:** idea stage frozen immediately before empirical work

| # | Idea | Novelty (provisional) | Cost to falsify | Status |
|---|---|---:|---:|---|
| 1 | **SemLibSQL-Γ / Guarded Abstraction Gap** | **5.5–6/10** | Low–Medium | **RECOMMENDED HIGH-RISK BET** |
| 2 | **TemporalSQL-Drift** — historical business-semantics benchmark | 6.5/10 | Low–Medium | **BEST BACKUP** |
| 3 | **DualSQL** — cross-task dual-control exploration | 7/10 | High | MOONSHOT BACKUP |
| 4 | DenoRepair — sparse result-feedback analytical SQL repair | 5/10 | Medium | DEPRIORITIZED |
| 5 | AutoSemanticView | 3/10 | — | ELIMINATED AS PAPER 1 |

## Active idea — SemLibSQL-Γ

**Research question:** Do realistic verified Text2SQL workloads contain a **Guarded Abstraction Gap**—reusable semantic operations that can only be safely recognized when equivalence is conditioned on local warehouse contracts—and can learned `(operator, guard)` pairs improve unseen composition beyond strong memory and generic library learning?

### Final novelty constraint

The method must beat or meaningfully differ from all of:

- Stitch;
- babble / LLMT;
- **E-Stitch (EGRAPHS 2026)**;
- ReGAL;
- predicate/colored-egraph conditional equality;
- **Alive-Infer-style precondition inference**;
- a composed baseline that combines library learning + conditional reasoning + separate guard inference;
- AgentSM/GATE-style Text2SQL memory.

If the composed PL baseline matches the joint method, drop the method novelty claim. If the Guarded Abstraction Gap itself is negligible, kill the entire direction.

### Required next step

Run **Gate A only** from:

`research/text2sql-agent/refine-logs/EXPERIMENT_PLAN.md`

No empirical pilot has been run in the idea stage.
