`.blueprint/04-tasks/T211-agent-protocol-v2-modules.md`
# T211: Upgrade Agent Protocol to v2 (`create_module`, `delete_module`, `add_import`, `remove_import`)

| Field | Value |
|---|---|
| Milestone | `M10` |
| Priority | `P0` |
| Depends on | T210 |
| Blocks | T212 |
| Acceptance families | V2-AGENT |
| Bounded areas | `crates/wolio-agent` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T211.md` |

## Objective
Extend `crates/wolio-agent` to support `protocol_version: 2` (`wolio.mutation.v2`),
adding multi-module plan operations (`create_module`, `delete_module`,
`add_import`, `remove_import`) and typed AST payloads for all `CORE_V2`
constructs while preserving backward-compatible `protocol_version: 1` handling.

## Authority
- Normative: [CORE_V2.md](../02-specifications/language/CORE_V2.md) (Section 6: V2 agent mutations), [LOCAL_PROTOCOL_V1.md](../02-specifications/protocol/LOCAL_PROTOCOL_V1.md), [MUTATION_SCHEMA_V1.md](../02-specifications/protocol/MUTATION_SCHEMA_V1.md), [APPROVAL_MATRIX_V1.md](../02-specifications/protocol/APPROVAL_MATRIX_V1.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-AGENT`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Protocol version negotiation (`crates/wolio-agent`):**
  - `protocol_version: 1`: continues to use hash domain `"wolio.mutation.v1"` and rejects V2 operation kinds/AST nodes.
  - `protocol_version: 2`: uses hash domain `"wolio.mutation.v2"` for plan hashes (`length_prefix("wolio.mutation.v2") || length_prefix(base_snapshot) || length_prefix(canonical_json(params))`).
- **New operation kinds in `protocol_version: 2`:**
  1. `create_module`: `target: null`, `expected_old_hash: null`, `payload: { "module": identifier, "path": "src/<name>.wol", "imports": [identifier, ...] }`.
     - `path` must be a normalized path under `src/` or `tests/` within `scope.paths` (rejecting `..`, symlinks, or overwriting an existing file/module with `WOLIO-SCOPE-001` / `WOLIO-RESOLVE-001`).
  2. `delete_module`: `target: "mod_<module_name>"` (or module snapshot hash), `expected_old_hash: "<sha256>"`, `payload: { "module": identifier }`.
     - Rejects deletion if any remaining module imports `module`.
  3. `add_import`: `target: "mod_<module_name>"`, `expected_old_hash: "<sha256>"`, `payload: { "module": identifier, "import": identifier }`.
  4. `remove_import`: `target: "mod_<module_name>"`, `expected_old_hash: "<sha256>"`, `payload: { "module": identifier, "import": identifier }`.
- **New V2 AST payload objects in `protocol_version: 2`:**
  - Declaration kind `"predicate"`: `{ "kind": "predicate", "name": identifier, "public": boolean, "parameters": [{ "name", "type" }], "body": expr }`.
  - Expression kind `"for"`: `{ "kind": "for", "item": identifier, "source": iter_source, "acc": identifier, "acc_type": type, "init": expr, "bound": expr, "invariant": expr, "lets": [{ "name", "value" }], "body": expr }` where `iter_source` is `{ "kind": "collection", "value": expr }` or `{ "kind": "range", "start": expr, "end": expr }`.
  - Expression kind `"format"`: `{ "kind": "format", "template": string, "args": [expr, ...] }`.
- **Approval tier assignment (`APPROVAL_MATRIX_V1.md` + `CORE_V2.md`):**
  - `create_module` (with private declarations only) and `add_import` / `remove_import` that satisfy `[architecture]` policy: `POST_REVIEW`.
  - `delete_module`, creating/deleting a module with `pub` declarations, or any `predicate` change affecting a `pub` contract or non-identical predicate: `HUMAN_REQUIRED`.
  - Any `add_import` violating `[architecture]` fails verification with `WOLIO-ARCH-001` and cannot be applied regardless of tier.

## Scope
In scope:
- `protocol_version: 2` envelope, `"wolio.mutation.v2"` canonical plan hashing, and V2 AST JSON deserialization/rendering in `wolio-agent`.
- `create_module`, `delete_module`, `add_import`, and `remove_import` staging, CAS journal recovery (including newly created/deleted module files), and tier classification.
- Maintaining 100% compatibility for `protocol_version: 1` requests.

Out of scope (owner):
- V2 anti-workaround analyzer checks on loops and constant folding (T212).

## Normative requirements
- `protocol_version: 2` adds `create_module`, `delete_module`, `add_import`, and `remove_import` and encodes `for`, `predicate`, `format`, and V2 types as structured AST objects—never raw source strings or text patches (CORE_V2, MUTATION_SCHEMA_V1).
- Scope validation checks `scope.paths` and `scope.symbols` before and after staging; path traversal (`..`), paths outside `src/`/`tests/`, and symlink escapes fail with `WOLIO-SCOPE-001` (LOCAL_PROTOCOL_V1).
- Crash recovery journal in `.wolio-log/` atomically rolls back or completes file creation/deletion for `create_module` and `delete_module` (LOCAL_PROTOCOL_V1).

## Acceptance criteria
Positive and boundary:
- [ ] `protocol_version: 2` plans can create a new module, add an allowed import, declare a `predicate` and bounded `for` loop operation, verify, apply, and finalize
- [ ] `create_module` with private symbols and allowed `add_import` classify as `POST_REVIEW`; public module exports or `delete_module` classify as `HUMAN_REQUIRED`
- [ ] Crash recovery during `create_module` / `delete_module` restores the workspace cleanly without orphaned files

Rejection and failure:
- [ ] `add_import` violating `wolio.toml` `[architecture]` fails plan verification with `WOLIO-ARCH-001`
- [ ] `create_module` outside `scope.paths` or with `..` path traversal fails with `WOLIO-SCOPE-001`
- [ ] Sending V2 operations (`create_module`, `for` AST) under `protocol_version: 1` fails schema validation before staging

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T211.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T211.md` with `V2-AGENT` multi-module mutation, tier classification, and journal recovery fixtures.

## Dependencies and open items
- Depends on: T210; blocks: T212.
- Open: none.

