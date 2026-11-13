`.blueprint/04-tasks/T072-formatter.md`
# T072: Implement canonical comment-preserving formatter and `wolio fmt`

| Field | Value |
|---|---|
| Milestone | `M07` |
| Priority | `P0` |
| Depends on | T071 |
| Blocks | T073 |
| Acceptance families | TOOL (formatting) |
| Bounded areas | formatter (`crates/wolio-format`), parser, CLI (`crates/wolio-cli`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T072.md` |

## Objective
Implement the canonical, idempotent, comment-preserving source formatter and
expose `wolio fmt` and read-only `wolio fmt --check`.

## Authority
- Normative: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) (M07 services), [CORE_V1.md](../02-specifications/language/CORE_V1.md), [SEMANTIC_MODEL.md](../02-specifications/SEMANTIC_MODEL.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `TOOL`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Comment-attaching/preserving canonical printer for all CORE_V1 grammar
  productions using four-space indentation and LF line endings.
- Post-format invariant check verifying preservation of the normalized semantic
  AST, declaration `sym_` IDs and contract clause identities.
- `wolio fmt` and read-only `wolio fmt --check` CLI commands (with `--json`
  support).

Out of scope (owner):
- Language Server Protocol implementation (T073); release packaging (T074).

## Normative requirements
- Formatter parses and prints canonical CORE_V1 syntax, preserves all `//`
  comments, uses four-space indentation and LF newlines, and is idempotent
  (DELIVERY_POLICY).
- Formatting preserves semantic AST structure, every declaration `sym_` ID and
  every normalized clause identity (DELIVERY_POLICY, SEMANTIC_MODEL).
- `wolio fmt --check` is strictly read-only, modifies no files, and exits
  non-zero when any discovered `.wol` file differs from canonical format
  (INTERFACE_CONTRACTS, DELIVERY_POLICY).

## Acceptance criteria
Positive and boundary:
- [ ] Formatting is idempotent across all conformance and trial `.wol` files (`fmt(fmt(x)) == fmt(x)`)
- [ ] All `//` comments, normalized AST nodes, declaration `sym_` IDs and clause identities are preserved before and after formatting
- [ ] CRLF inputs normalize to LF with four-space indentation

Rejection and failure:
- [ ] `wolio fmt --check` exits non-zero on unformatted source without modifying any file on disk
- [ ] Syntax-invalid files fail with `WOLIO-SYNTAX-001` without corrupting or truncating the file

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p wolio-cli -- fmt --check --project <tool-fixture-project> --json
```
Evidence: `.execution/reports/T072.md` with idempotence, comment-preservation and `--check` fixtures.

## Dependencies and open items
- Depends on: T071; blocks: T073.
- Open: none.

