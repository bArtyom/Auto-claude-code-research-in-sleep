# Round 3 Pipeline Summary

**Problem:** Find a Text2SQL × agent research direction that remains meaningfully novel after current 2025–2026 literature, admits cheap falsification, and does not rely on adding another generic agent loop.  
**Date:** 2026-08-23  
**Final method thesis:** Learn a warehouse-specific semantic program library from verified SQL experience and test whether induced operators enable held-out composition beyond episodic query memory.  
**Final verdict:** **READY FOR CHEAP PILOT / FORMAL CROSS-MODEL REVIEW UNAVAILABLE**

## Workflow executed

```text
research-lit
   |
   +-- DreamSQL deep prior-art review
   +-- Result2SQL deep prior-art review
   +-- AutoSemanticView deep prior-art review
        |
        v
novelty-check
        |
        +-- kill naive Result2SQL claim
        +-- kill/merge naive AutoSemanticView claim
        +-- narrow DreamSQL to SemLibSQL
        |
        v
research-review fallback
        |
        +-- internal adversarial review only
        +-- formal independent reviewer unavailable
        |
        v
research-refine
        |
        v
FINAL_PROPOSAL.md
        |
        v
EXPERIMENT_PLAN.md
```

## Deliverables

- `DREAMSQL_LITERATURE_REVIEW.md` — library learning, agent skills, Text2SQL memory, surviving novelty.
- `RESULT2SQL_LITERATURE_REVIEW.md` — provenance, why-not, PBE, interactive repair; direct 2013 collision documented.
- `AUTOSEMANTICVIEW_LITERATURE_REVIEW.md` — semantic layers, auto semantic-view generation, materialized-view/self-driving DB overlap.
- `LITERATURE_CORRECTIONS.md` — corrects the earlier SPARTA mischaracterization.
- `NOVELTY_ADJUDICATION.md` — claim-by-claim novelty decision.
- `CRITICAL_REVIEW.md` — adversarial internal review, explicitly provisional.
- `FINAL_PROPOSAL.md` — focused SemLibSQL thesis.
- `EXPERIMENT_PLAN.md` — staged falsification plan.

## Contribution snapshot

### Dominant contribution

**SemLibSQL**: semantics-aware program-library induction from verified Text2SQL experience.

The core scientific delta is:

```text
verified query history
        |
        |  memory baselines keep/retrieve it
        v
semantic normalization + canonicalization
        |
        v
new parameterized semantic operators
        |
        v
held-out compositions
```

### Explicitly rejected complexity

Not in Paper 1:

- DenoRepair/result correction;
- automatic semantic view/materialized view creation;
- active schema exploration;
- bitemporal long-term governance;
- multi-agent debate;
- RL;
- proof systems;
- large autonomous platform.

## Novelty decisions

| Direction | Decision | Rationale |
|---|---|---|
| **SemLibSQL / DreamSQL** | **PROCEED WITH CAUTION** | survives only as semantics-aware typed abstraction induction beyond memory |
| **DenoRepair / revised Result2SQL** | **KEEP SEPARATE** | naive tuple-feedback refinement has old direct prior art; revised complex structural repair remains plausible |
| **AutoSemanticView** | **KILL AS STANDALONE** | semantic layers, automatic semantic-view products and materialized-view selection make generic claim crowded |
| **Semantic Promotion** | **DEFER** | possible Paper-2 systems problem after SemLibSQL works |

## Must-prove claims

1. Semantic canonicalization finds reusable structures that syntactic library mining misses.
2. Automatically induced primitives improve held-out composition beyond strong verified-query memory.
3. Learned operators are semantically coherent and parameterized, not frequent AST fragments.
4. Library reuse has low negative transfer.

## First runs to launch

1. Build a 200–500-query corpus with 8–12 controlled recurring semantic motifs and diverse SQL realizations.
2. Compare token/AST/Stitch-like abstraction mining with semantic normalization/canonicalization.
3. If Gate A passes, create leakage-controlled held-out motif compositions.
4. Compare stateless generation, raw query retrieval, AgentSM-like structured memory, Crystallization-like verified memory, syntactic learned library, oracle/manual library, and SemLibSQL.

## Kill conditions

Stop before large implementation if:

- syntactic mining matches semantic canonicalization;
- strong query memory matches learned-library held-out composition;
- induced primitives are mostly syntactic;
- automatic library is far below a small manual semantic library;
- real/enterprise-like workload does not reproduce controlled gains;
- negative transfer creates meaningful silent-error risk.

## Formal review state

The repository skills require an identity-bearing independent reviewer for formal novelty/research acceptance. This connector session does not expose that backend.

```yaml
review_independence: unavailable
acceptance_status: provisional
formal_review_gate: REVIEW_UNAVAILABLE
```

No reviewer identity/thread/receipt has been invented.

## Best next action

**Run only the Phase-A abstraction feasibility pilot.**

Do not begin a full SQL agent architecture or model-training program until the semantic-normalization and held-out-composition gates show a positive signal.