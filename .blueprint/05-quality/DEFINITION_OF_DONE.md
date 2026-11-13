`.blueprint/05-quality/DEFINITION_OF_DONE.md`
# Definition of done

For a task: its contract and assigned ACCEPTANCE_MATRIX families pass, common
Rust checks pass, regressions are covered, relevant user docs are accurate, and
`.execution/reports/Tnnn.md` records actual commands and evidence. Unrun checks
remain unrun. No undocumented semantics, unrelated changes or placeholder success.

For a milestone: all constituent tasks and prerequisites are accepted, including
failure/boundary cases. Deferred extensions are not hidden release blockers.

For the trial: M01–M03, three examples and change cycles, fresh-user instructions,
and pilot results. For V1: every M01–M07 gate, including expanded evaluation and
toolchain distribution. Refer to RELEASE_CRITERIA and ACCEPTANCE_MATRIX.
