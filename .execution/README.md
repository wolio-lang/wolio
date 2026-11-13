# Execution ledger

Blueprint defines intended behavior; this ledger records actual state and evidence.
`baseline-acceptance.md` records freeze acceptance, `current-state.md` actual
capabilities, `task-status.md` progress, `blockers/` concrete impediments,
`decisions/` execution decisions, and `reports/` task evidence.

Statuses: PLANNED, READY, IN_PROGRESS, BLOCKED, DONE. DONE requires evidence.
Update current state and task status when work changes. Reports identify scope,
changes, checks and follow-up. Design completion is not implementation completion.

The ledger exists before coding because design audits, baseline review and setup
are real activities worth recording. PREP-nnnn reports describe those activities;
Tnnn reports describe compiler/toolchain implementation. A completed PREP report
does not make any T task complete. Historical reports remain historical; owner
acceptance is a separate decision, never inferred from a documentation request.
