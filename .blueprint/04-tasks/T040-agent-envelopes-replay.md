`.blueprint/04-tasks/T040-agent-envelopes-replay.md`
# T040: Implement local agent JSONL transport, query methods and replay log

| Field | Value |
|---|---|
| Milestone | `M04` |
| Priority | `P0` |
| Depends on | T034 |
| Blocks | T041 |
| Acceptance families | AGENT (versions, query, replay) |
| Bounded areas | agent (`crates/wolio-agent`), CLI (`crates/wolio-cli`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T040.md` |

## Objective
Serve the local stdin/stdout JSON-lines protocol (`wolio agent`) with request
validation, snapshot-checked `inspect`/`impact`/`verify` queries, and
append-only idempotent request replay under `.wolio-log/`.

## Authority
- Normative: [LOCAL_PROTOCOL_V1.md](../02-specifications/protocol/LOCAL_PROTOCOL_V1.md), [MUTATION_SCHEMA_V1.md](../02-specifications/protocol/MUTATION_SCHEMA_V1.md) (canonical encoding and limits), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `AGENT`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- `wolio agent` local JSON-lines loop over stdin/stdout (no network listener).
- Request envelope validation (`protocol_version: 1`, `request_id`, `method`,
  `base_snapshot`, `params`) and 1 MiB line limit.
- Snapshot-checked `inspect`, `impact` and `verify` protocol methods reusing
  M03 services.
- Append-only `.wolio-log/` request/response ledger and `request_id` replay/
  conflict detection.

Out of scope (owner):
- `plan` payload validation, AST rendering and canonical diffs (T041); scope and
  approval tier enforcement (T042); staged `apply` and crash recovery (T043);
  `finalize`, bounded repair and review ledger (T044).

## Normative requirements
- Transport is local UTF-8 JSON-lines over stdin/stdout under the invoking OS
  user; no HTTP server or remote auth exists in V1 (LOCAL_PROTOCOL_V1).
- Every request requires `protocol_version: 1`, non-empty `request_id`,
  `method` (`inspect`, `impact`, `plan`, `apply`, `verify`, `finalize`),
  `base_snapshot` and object `params`; lines over 1 MiB UTF-8, duplicate JSON
  keys, floats, unknown fields, unknown versions or unknown methods fail before
  any mutation (LOCAL_PROTOCOL_V1, MUTATION_SCHEMA_V1).
- Every response repeats `protocol_version` and `request_id` and contains
  exactly one of `result` or `error` (`code`, `message`, `diagnostics`)
  (LOCAL_PROTOCOL_V1).
- Query and verification requests require `base_snapshot` to match the current
  project snapshot and fail with `WOLIO-STALE-001` on mismatch
  (LOCAL_PROTOCOL_V1, EVIDENCE_MODEL).
- Append-only JSONL records are persisted under `.wolio-log/`, excluded from
  the source snapshot hash, and omit secret values. Repeating a `request_id`
  with an identical canonical payload replays the recorded response; repeating a
  `request_id` with a changed payload fails explicitly (LOCAL_PROTOCOL_V1,
  SEMANTIC_MODEL).

## Acceptance criteria
Positive and boundary:
- [ ] Valid `inspect`, `impact` and `verify` requests over JSONL return deterministic results matching CLI services
- [ ] Repeated `request_id` with identical payload replays the recorded response from `.wolio-log/`
- [ ] `.wolio-log/` entries do not alter the project source snapshot hash

Rejection and failure:
- [ ] Unknown `protocol_version`, unknown `method`, unknown fields, duplicate keys or lines > 1 MiB are rejected before mutation
- [ ] Mismatched `base_snapshot` on queries fails with `WOLIO-STALE-001`
- [ ] Repeated `request_id` with a modified payload is rejected

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
Evidence: `.execution/reports/T040.md` with AGENT envelope and replay fixtures.

## Dependencies and open items
- Depends on: T034; blocks: T041.
- Open: none.

