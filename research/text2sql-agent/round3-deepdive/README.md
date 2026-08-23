# Round 3 — Deep Literature Adjudication

This directory is the third Text2SQL-agent idea-discovery round. It follows the repository workflow:

`research-lit -> idea/claim extraction -> novelty-check -> research-review -> research-refine -> experiment-plan`

## Reading order

1. `DREAMSQL_LITERATURE_REVIEW.md`
2. `RESULT2SQL_LITERATURE_REVIEW.md`
3. `AUTOSEMANTICVIEW_LITERATURE_REVIEW.md`
4. `LITERATURE_CORRECTIONS.md`
5. `NOVELTY_ADJUDICATION.md`
6. `CRITICAL_REVIEW.md`
7. `FINAL_PROPOSAL.md`
8. `EXPERIMENT_PLAN.md`
9. `PIPELINE_SUMMARY.md`

## Current decision

The surviving first-paper candidate is **SemLibSQL (project codename DreamSQL)**: learn a typed warehouse-specific semantic program library from verified SQL histories and test true held-out composition beyond episodic memory.

Two earlier directions were narrowed by prior art:

- naive Result2SQL is not novel because tuple-feedback query refinement predates LLMs; the salvage is a separate **DenoRepair** line for sparse heterogeneous denotational constraints + provenance-guided structural repair + collateral-damage evaluation;
- naive AutoSemanticView is crowded by semantic-layer automation and classic workload materialization; retain **Semantic Promotion** only as a future extension after learned semantic operators are proven useful.

## Review caveat

The connector session used for this round does not expose the independent Codex/manual reviewer required by ARIS for a formal acceptance receipt. `CRITICAL_REVIEW.md` is explicitly an internal red-team artifact and the proposal remains provisional until that external gate is run.