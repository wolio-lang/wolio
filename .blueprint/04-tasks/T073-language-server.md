`.blueprint/04-tasks/T073-language-server.md`
# T073: Implement incremental Language Server Protocol (`wolio lsp`) and safe rename

| Field | Value |
|---|---|
| Milestone | `M07` |
| Priority | `P0` |
| Depends on | T072 |
| Blocks | T074 |
| Acceptance families | TOOL (editor) |
| Bounded areas | LSP (`crates/wolio-lsp`), semantic/query, CLI (`crates/wolio-cli`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T073.md` |

## Objective
Provide `wolio lsp` over stdio supporting incremental document sync,
diagnostics, hover, go-to-definition, find-references and version-checked
workspace rename with explicit old/new `sym_` identity mappings.

## Authority
- Normative: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) (M07 services), [SEMANTIC_MODEL.md](../02-specifications/SEMANTIC_MODEL.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `TOOL`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- `wolio lsp` JSON-RPC server lifecycle (`initialize`, `initialized`,
  `shutdown`, `exit`).
- In-memory overlay for open/unsaved document buffers and incremental document
  updates without mutating disk files.
- Diagnostics, hover (signatures, contracts, nominal types, `sym_` IDs),
  go-to-definition and find-references backed by the shared semantic snapshot
  model.
- Workspace rename emitting reviewed versioned edits plus explicit old/new
  `sym_` identity provenance mappings, rejecting stale document versions.

Out of scope (owner):
- Release binary archive packaging and installation verification (T074).

## Normative requirements
- LSP supports `initialize`/`shutdown`, incremental document updates,
  diagnostics, hover, definition, references and rename using the same semantic
  snapshot model as the CLI and agent protocol (DELIVERY_POLICY,
  IMPLEMENTATION_SPECIFICATION_V1).
- Unsaved editor buffers never overwrite disk files or pollute on-disk project
  caches (DELIVERY_POLICY, ACCEPTANCE_MATRIX).
- Rename emits a reviewed workspace edit including explicit old/new `sym_`
  identity mappings (moving an operation and its implementation together and
  updating resolved references); stale document versions reject edit generation
  or application (DELIVERY_POLICY, SEMANTIC_MODEL).

## Acceptance criteria
Positive and boundary:
- [ ] `initialize`/`shutdown`, incremental edits, diagnostics, hover, definition and references match compiler semantic queries
- [ ] Workspace rename updates all resolved references across modules and attaches the old/new `sym_` identity mapping
- [ ] Unsaved buffer edits remain isolated in memory with zero disk writes

Rejection and failure:
- [ ] Rename requests against stale document versions or producing duplicate/reserved names are rejected
- [ ] Attempting to rename builtin types/constructors or read-only external package symbols is rejected

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
Evidence: `.execution/reports/T073.md` with LSP session, unsaved-buffer isolation and stale-version rename fixtures.

## Dependencies and open items
- Depends on: T072; blocks: T074.
- Open: none.

