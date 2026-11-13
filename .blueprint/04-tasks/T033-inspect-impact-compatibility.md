`.blueprint/04-tasks/T033-inspect-impact-compatibility.md`
# T033: Implement `inspect`, `impact` and contract compatibility analysis

| Field | Value |
|---|---|
| Milestone | `M03` |
| Priority | `P0` |
| Depends on | T032 |
| Blocks | T034 |
| Acceptance families | QUERY |
| Bounded areas | semantic/query (`crates/wolio-semantic`), CLI (`crates/wolio-cli`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T033.md` |

## Objective
Provide read-only `wolio inspect` and `wolio impact` commands returning
deterministic semantic graphs, reverse-reachability paths and old/new contract
compatibility classifications.

## Authority
- Normative: [SEMANTIC_MODEL.md](../02-specifications/SEMANTIC_MODEL.md), [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `QUERY`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- `wolio inspect [symbol-id]` over the T023 semantic graph in human and `--json`
  modes.
- Contract and declaration compatibility classifier between two snapshots.
- Reverse-reachability impact traversal over typed edges with edge-path
  explanations.
- `wolio impact --before <project-dir> --after <project-dir>` CLI command.

Out of scope (owner):
- Agent protocol `inspect`/`impact` JSONL transport and provenance links from
  structured rename/move plans (T040–T041).
- LSP workspace references and rename edits (T073).

## Normative requirements
- `inspect [symbol-id]` returns sorted nodes, sorted edges and the snapshot
  hash; omitting `symbol-id` returns the whole package graph. `inspect
  --register-ids` is not supported. Neither `inspect` nor `impact` writes to the
  project (SEMANTIC_MODEL, INTERFACE_CONTRACTS).
- `impact --before <project-dir> --after <project-dir>` compares two explicit
  snapshots and reports semantic declaration/clause changes, compatibility
  classifications and reverse-reachable callers/tests with explicit typed edge
  paths (`imports`, `calls`, `depends_on`, `returns`, `uses_type`,
  `has_contract`, `has_effect`, `tests`), without claiming reachability alone
  proves behavioral breakage (SEMANTIC_MODEL, INTERFACE_CONTRACTS).
- Read-only snapshot comparison reports rename/move as removal plus addition and
  never infers lineage from similarity; changing package identity (`local:` to
  `git:`) changes all declaration IDs and requires an explicit impact report
  (SEMANTIC_MODEL, INTERFACE_CONTRACTS).
- Compatibility compares signatures, normalized predicates and
  error/effect/dependency sets: removed public symbols, changed nominal types or
  arity, added errors/effects/dependencies, and removed error variants from a
  public enum are breaking. V1 proves predicate compatibility only when
  normalized predicates are identical; any other predicate edit is classified
  `unknown`, never automatically compatible (CONTRACT_DESIGN).

## Acceptance criteria
Positive and boundary:
- [ ] `inspect` outputs identical sorted nodes, edges and snapshot hash across runs
- [ ] `impact` reports reverse-reachable callers and tests with typed edge paths
- [ ] Edits preserving normalized predicates, signatures and sets classify as compatible

Rejection and failure:
- [ ] Removed public API, changed nominal type/signature/arity, added error/effect/dependency or removed public error variant is classified breaking
- [ ] Non-identical `requires`/`ensures` predicate edits classify as `unknown`, never compatible
- [ ] Package identity change without impact or implicit rename transfer in read-only comparison is rejected

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p wolio-cli -- inspect --project <query-fixture-project> --json
cargo run -p wolio-cli -- impact --before <before-dir> --after <after-dir> --json
```
Evidence: `.execution/reports/T033.md` with the QUERY fixture list.

## Dependencies and open items
- Depends on: T032; blocks: T034.
- Open: none.

