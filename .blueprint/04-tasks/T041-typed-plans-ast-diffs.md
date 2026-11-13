`.blueprint/04-tasks/T041-typed-plans-ast-diffs.md`
# T041: Implement typed mutation plans, canonical AST diffs and identity links

| Field | Value |
|---|---|
| Milestone | `M04` |
| Priority | `P0` |
| Depends on | T040 |
| Blocks | T042 |
| Acceptance families | AGENT (plan, raw-text rejection, old/new ID mapping) |
| Bounded areas | agent (`crates/wolio-agent`), semantic |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T041.md` |

## Objective
Validate typed AST mutation plans under `MUTATION_SCHEMA_V1`, render canonical
comment-preserving Wolio source, compute canonical AST diff hashes and produce
explicit old/new `sym_` provenance mappings for renames and moves.

## Authority
- Normative: [LOCAL_PROTOCOL_V1.md](../02-specifications/protocol/LOCAL_PROTOCOL_V1.md), [MUTATION_SCHEMA_V1.md](../02-specifications/protocol/MUTATION_SCHEMA_V1.md), [SEMANTIC_MODEL.md](../02-specifications/SEMANTIC_MODEL.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md)
- Decision: [ADR-007](../01-architecture/decisions/ADR-007-deterministic-identity-and-gated-automation.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `AGENT`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Full `MUTATION_SCHEMA_V1` validation for `create_declaration`,
  `replace_implementation`, `replace_contract`, `rename_symbol`, `move_symbol`,
  `add_test` and `delete_symbol`.
- Plan hash (`wolio.mutation.v1`) and canonical AST diff (`wolio.ast-diff.v1`)
  computation.
- Deterministic AST-to-source renderer preserving unrelated declarations and
  comments, followed by frontend/semantic re-checking.
- Atomic reference updates and explicit old/new `sym_` provenance pairs for
  `rename_symbol` and `move_symbol`.

Out of scope (owner):
- Scope, capability and approval-tier enforcement (T042); staged workspace lock
  and transactional `apply`/journal recovery (T043); analyzer rules, review
  disposition and `finalize` (T044).

## Normative requirements
- `plan` requires non-empty `intent`, `operations` (1..256), `scope` (`paths`,
  `symbols`) and `capabilities`, with maximum AST depth 128. Every operation has
  `kind`, `target` (omitted only for `create_declaration`), `expected_old_hash`
  (null only for create) and typed `payload` (MUTATION_SCHEMA_V1).
- Payloads encode CORE_V1 AST nodes as tagged JSON objects; raw source strings,
  patches, byte offsets, arbitrary file paths, source spans, unknown tags or
  fields, duplicate set entries and high-level aliases (`create_operation`) are
  rejected before staging (LOCAL_PROTOCOL_V1, MUTATION_SCHEMA_V1).
- `create_declaration` requires a vacant semantic key in an existing module;
  `add_test` targets the tested operation and requires a vacant test identity;
  other operations verify `expected_old_hash` against the target's normalized
  declaration hash (MUTATION_SCHEMA_V1).
- The renderer outputs canonical CORE_V1 syntax (rendering Int `MIN` as unary
  minus over `9223372036854775808` and `unit` as `unit()`), preserves unrelated
  source and comments, never invents `requires`/`ensures` or widens
  `effects`/`dependencies`/`errors`, and re-runs parse/semantic checks on the
  resulting graph (LOCAL_PROTOCOL_V1, MUTATION_SCHEMA_V1).
- Plan hash uses length-prefixed `wolio.mutation.v1`, `base_snapshot` and
  canonical JSON `params`. Canonical AST diff is a byte-sorted array of
  `{id, before, after}` normalized AST entries hashed with `wolio.ast-diff.v1`
  (MUTATION_SCHEMA_V1, INTERFACE_CONTRACTS).
- `rename_symbol` and `move_symbol` move an operation and its implementation
  together, update resolved references across the plan, and return explicit
  old/new `sym_` identity pairs bound to base/result snapshots
  (LOCAL_PROTOCOL_V1, MUTATION_SCHEMA_V1, SEMANTIC_MODEL).

## Acceptance criteria
Positive and boundary:
- [ ] Typed payload round-trips succeed for every CORE_V1 declaration, contract, pattern, type and expression production
- [ ] Canonical plan hashes and `wolio.ast-diff.v1` diff vectors are deterministic and independent of request ID or formatting
- [ ] `rename_symbol` and `move_symbol` update references atomically and emit explicit old/new `sym_` links
- [ ] Renderer preserves unrelated declarations and comments in modified modules

Rejection and failure:
- [ ] Raw source strings, text patches, byte offsets, spans, unknown tags/fields or > 256 operations / > 128 AST depth are rejected
- [ ] Occupied semantic keys on create, mismatched `expected_old_hash`, conflicting operations in one plan or unavailable imports are rejected

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
Evidence: `.execution/reports/T041.md` with typed payload round-trip, rejection and canonical diff fixtures.

## Dependencies and open items
- Depends on: T040; blocks: T042.
- Open: none.

