# Text2SQL × Agent Idea Candidates — Final Shortlist

**Date:** 2026-08-23  
**Status:** idea stage frozen before experiments

| # | Idea | Novelty | Feasibility | Status |
|---|---|---|---|---|
| 1 | **SemLibSQL-Γ — contract-conditioned semantic library learning** | 7/10 provisional | Medium | **RECOMMENDED** |
| 2 | TemporalSQL-Drift — bitemporal business-semantics benchmark | 6.5/10 | High | BACKUP |
| 3 | DualSQL — current-task + future-task database exploration | 7/10 | Medium-Low | BACKUP / MOONSHOT |
| 4 | DenoRepair — sparse denotational-feedback SQL repair | 5/10 | High | DEPRIORITIZED |
| 5 | AutoSemanticView | 3/10 | Medium | ELIMINATED AS PAPER 1 |

## Active Idea: #1 — SemLibSQL-Γ

**Hypothesis:** Verified SQL histories contain reusable semantic motifs whose implementations differ so much syntactically that generic library learners miss them; conditioning equivalence on warehouse contracts and attaching applicability conditions to learned operators can recover these motifs safely and improve held-out composition beyond structured memory.

**Key novelty constraint:** directly beat strong babble/ReGAL/Stitch-style library-learning baselines. If a fixed-theory generic learner matches the proposed method, kill the SQL-specific mechanism.

**Closest prior work:** DreamCoder, Stitch, babble/LLMT, ReGAL, U-semiring/VeriEQL, AgentSM, GATE, semantic-layer-mediated Text2SQL, Snowflake Semantic View Autopilot.

**Required next step:** Gate-A controlled experiment from `research/text2sql-agent/refine-logs/EXPERIMENT_PLAN.md`.

**No empirical pilot has been run.**
