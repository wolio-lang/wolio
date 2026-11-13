`.blueprint/04-tasks/T043-transactional-apply-recovery.md`
# T043: Implement staged verification, CAS publication and journal crash recovery

| Field | Value |
|---|---|
| Milestone | `M04` |
| Priority | `P0` |
| Depends on | T042 |
| Blocks | T044 |
| Acceptance families | AGENT (crash recovery, concurrent-edit preservation) |
| Bounded areas | agent (`crates/wolio-agent`), filesystem transaction/journal adapter |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T043.md` |

## Objective
Apply approved mutation plans atomically via an isolated staging workspace,
exclusive project lock, compare-and-swap snapshot check and write-ahead journal
recovery that never discards concurrent user edits.

## Authority
- Normative: [LOCAL_PROTOCOL_V1.md](../02-specifications/protocol/LOCAL_PROTOCOL_V1.md), [APPROVAL_MATRIX_V1.md](../02-specifications/protocol/APPROVAL_MATRIX_V1.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `AGENT`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Isolated temporary staging workspace for rendering and pre-publication
  verification (`check` and `test` with explicitly granted capabilities).
- Exclusive project mutation lock and compare-and-swap check against
  `base_snapshot` and approval state before publication.
- Write-ahead journal recording old/new content hashes, provenance and atomic
  `POST_REVIEW` pending review record creation.
- Deterministic startup/pre-mutation crash recovery (roll-forward or rollback)
  preserving concurrent user edits.

Out of scope (owner):
- Bounded repair loop, owner review disposition (`accepted`/`rejected`),
  `finalize` and M04 analyzer rules (T044).

## Normative requirements
- `apply` stages changes in an isolated temporary workspace, validates scope
  before and after staging, runs full verification in staging (granting tests
  only explicitly authorized execution capabilities), acquires an exclusive
  project mutation lock, and re-verifies the live workspace snapshot and
  approval before compare-and-swap publication (LOCAL_PROTOCOL_V1).
- If the workspace snapshot or approval changed since `base_snapshot`, `apply`
  aborts with `WOLIO-STALE-001` without modifying any working-tree file
  (LOCAL_PROTOCOL_V1, EVIDENCE_MODEL).
- A write-ahead journal records old and new file content hashes, plan/diff
  provenance and publication steps. For `POST_REVIEW` plans, the pending review
  record under `.wolio-log/` is written atomically in the same recoverable
  transaction; missing review record blocks `POST_REVIEW` publication
  (LOCAL_PROTOCOL_V1, APPROVAL_MATRIX_V1).
- After an interruption or crash at any staging, journal or publication step,
  recovery completes or restores the entire managed transaction before another
  mutation can run, and never overwrites or discards concurrent/unrelated user
  edits (LOCAL_PROTOCOL_V1).

## Acceptance criteria
Positive and boundary:
- [ ] Valid `AUTO`, `POST_REVIEW` and approved `HUMAN_REQUIRED` plans verify in staging and publish atomically
- [ ] `POST_REVIEW` publication atomically records a `pending` review entry in `.wolio-log/`
- [ ] Fault injection at every journal and publication boundary recovers cleanly to a consistent snapshot on restart

Rejection and failure:
- [ ] Concurrent user edits or changed base snapshot abort `apply` with `WOLIO-STALE-001` and preserve user edits intact
- [ ] Failed verification (`check` or `test`) in staging aborts `apply` with diagnostics and leaves the workspace untouched
- [ ] Missing `POST_REVIEW` review record or consumed single-use approval blocks publication

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Fault-inject every journal/publication boundary and verify restart recovery and
concurrent-edit preservation.
Evidence: `.execution/reports/T043.md` with fault-injection and concurrent-edit fixtures.

## Dependencies and open items
- Depends on: T042; blocks: T044.
- Open: none.

