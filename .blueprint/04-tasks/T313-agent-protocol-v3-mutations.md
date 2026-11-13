`.blueprint/04-tasks/T313-agent-protocol-v3-mutations.md`
# T313: Upgrade Agent Protocol to v3 (`property`, `resource`, `with_mock`, `Secret` rules)

| Field | Value |
|---|---|
| Milestone | `M15` |
| Priority | `P0` |
| Depends on | T312 |
| Blocks | T314 |
| Acceptance families | V3-FUZZ, AGENT |
| Bounded areas | `crates/wolio-agent`, `crates/wolio-analyzer` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T313.md` |

## Objective
Upgrade `crates/wolio-agent` to support `protocol_version: 3` (`wolio.mutation.v3`)
with typed AST mutation payloads for `property`, `resource`, `modifies`, `old`,
`transaction`, and `with_mock`, enforce `HUMAN_REQUIRED` approval on any
`modifies` frame, `Secret<T>` sink, or `[capabilities]` change, and require
`wolio verify --fuzz` to pass before `apply` and `finalize`.

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md), [LOCAL_PROTOCOL_V1.md](../02-specifications/protocol/LOCAL_PROTOCOL_V1.md), [MUTATION_SCHEMA_V1.md](../02-specifications/protocol/MUTATION_SCHEMA_V1.md), [APPROVAL_MATRIX_V1.md](../02-specifications/protocol/APPROVAL_MATRIX_V1.md), [EXTENSION_BOUNDARIES.md](../02-specifications/EXTENSION_BOUNDARIES.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 1 & Section 2: `wolio-agent`)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-FUZZ` and `AGENT`, M15 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Protocol version negotiation (`crates/wolio-agent`):**
  - Supports `protocol_version: 1` (`"wolio.mutation.v1"`), `protocol_version: 2` (`"wolio.mutation.v2"`), and `protocol_version: 3` (`"wolio.mutation.v3"`).
  - Plan hash under v3: `SHA-256(length_prefix("wolio.mutation.v3") || length_prefix(base_snapshot) || length_prefix(canonical_json(params)))`.
- **New V3 AST payload schemas in `protocol_version: 3`:**
  1. Declaration kind `"resource"`: `{ "kind": "resource", "name": identifier, "public": boolean }`.
  2. Declaration kind `"property"`: `{ "kind": "property", "name": identifier, "target_operation": identifier, "parameters": [{ "name", "type" }], "where": expr | null, "lets": [{ "name", "value" }], "body": expr }`.
  3. Parameter mode `"resource"`: `{ "name": identifier, "type": type, "mode": "value" | "resource" }` (defaults to `"value"`).
  4. Operation contract field `"modifies"`: `[identifier, ...]`.
  5. Expression kind `"old"`: `{ "kind": "old", "expr": expr }`.
  6. Expression kind `"transaction"`: `{ "kind": "transaction", "handle": identifier, "lets": [{ "name", "value" }], "body": expr }`.
  7. Expression kind `"with_mock"`: `{ "kind": "with_mock", "handlers": [{ "target": qualified_ident, "parameters": [...], "return_type": type, "lets": [...], "body": expr }], "lets": [...], "body": expr }`.
- **V3 Approval Tier & Verification Gate rules (`APPROVAL_MATRIX_V1.md` + `V3-TASKS.md`):**
  - Any mutation that adds, removes, or modifies a `modifies [...]` clause, adds `Secret::Use` or changes a `Secret<T>` parameter/sink, weakens or deletes an existing `property` declaration, or alters `[capabilities]` is classified as `HUMAN_REQUIRED`.
  - Adding a new `property` declaration or strengthening a private implementation inside an existing `transaction` frame without changing contracts/effects is classified as `AUTO` (or `POST_REVIEW` if adding new tests/mocks).
  - For all `[language] version = "3"` workspaces, `plan`, `apply`, and `finalize` automatically run `wolio verify --fuzz`; any `WOLIO-FUZZ-001` counterexample blocks `apply` and `finalize` and returns the shrunk counterexample in the structured diagnostic repair response.
  - All `.wolio-log/` request/response/journal files redact any `Secret<T>` test values as `"<REDACTED:Secret>"`.

## Scope
In scope:
- `protocol_version: 3` envelope, `"wolio.mutation.v3"` plan hash domain, and V3 AST JSON deserialization/rendering in `wolio-agent`.
- `HUMAN_REQUIRED` tier enforcement on `modifies`, `Secret<T>` sinks, `property` weakening/deletion, and `[capabilities]`.
- Mandatory `wolio verify --fuzz` execution during `wolio-agent` `plan`/`apply`/`finalize` for V3 workspaces.

Out of scope (owner):
- V3 Native LLVM, `wolio fmt`, and `wolio lsp` parity (T314); full W01–W15 benchmark and V3 release (T315).

## Normative requirements
- `protocol_version: 3` requires structured AST objects for all V3 constructs (`property`, `resource`, `modifies`, `old`, `transaction`, `with_mock`) and rejects raw source strings or text diffs (MUTATION_SCHEMA_V1, V3-TASKS).
- Modifying `modifies` frames, `Secret<T>` sinks, `[capabilities]`, or weakening/removing a `property` requires `HUMAN_REQUIRED` approval (`WOLIO-SCOPE-001` if attempted without valid human approval token) (APPROVAL_MATRIX_V1, V3-TASKS).
- In V3 projects, `apply` and `finalize` require `wolio verify --fuzz` to pass with zero `WOLIO-FUZZ-001` counterexamples (V3-TASKS).

## Acceptance criteria
Positive and boundary:
- [ ] `protocol_version: 3` plans can add `property` declarations, `with_mock` tests, `resource` parameters, `modifies` clauses, `old(...)` postconditions, and `transaction (db) { ... }` bodies
- [ ] When an AI agent submits a hardcoded implementation in `protocol_version: 3`, `plan`/`apply` rejects it with `WOLIO-FUZZ-001` and returns the shrunk counterexample so the agent can repair the algorithm in the next turn
- [ ] Backward compatibility for `protocol_version: 1` and `2` envelopes is preserved

Rejection and failure:
- [ ] Attempting to change `modifies [...]`, add `Secret::Use`, or delete/weaken a `property` without a `HUMAN_REQUIRED` approval token fails with `WOLIO-SCOPE-001`
- [ ] Sending V3 AST payloads under `protocol_version: 1` or `2` fails schema validation before staging

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T313.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T313.md` with Agent Protocol v3 mutation, tier escalation, and fuzz-repair loop fixtures.

## Dependencies and open items
- Depends on: T312; blocks: T314.
- Open: none.

