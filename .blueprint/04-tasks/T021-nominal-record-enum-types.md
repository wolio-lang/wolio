`.blueprint/04-tasks/T021-nominal-record-enum-types.md`
# T021: Check nominal, record and enum types

| Field | Value |
|---|---|
| Milestone | `M02` |
| Priority | `P0` |
| Depends on | T020 |
| Blocks | T022 |
| Acceptance families | SEM (constructors, types) |
| Bounded areas | `crates/wolio-types`, semantic |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T021.md` |

## Objective
Statically type every non-generic CORE_V1 expression and declaration —
nominal, record, enum, operators, lets, blocks, branches and operation calls —
rejecting mismatches with structured diagnostics.

## Authority
- Normative: [CORE_V1.md](../02-specifications/language/CORE_V1.md) (types and values), [SEMANTIC_MODEL.md](../02-specifications/SEMANTIC_MODEL.md) (interned types), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md) (code registry)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `SEM`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope: builtin `Bool`, `Int`, `String`, `Unit`; nominal declarations,
construction and `.value`; records; enums and their constructors; operator,
let, block, if-branch and call typing; recursive value-type rejection.

Out of scope (owner):
- `Option`/`Result`, other builtin generics, `unit()`, match and Result-handling
  obligations (T022); persistent identity hashing (T023); entry signature
  checks at run (T024); contract predicate typing (T030); error/effect sets (T031).

## Normative requirements
- `type CustomerId = Int;` creates a nominal type, not an alias; `CustomerId(1)`
  constructs; `.value` extracts the representation; distinct nominal types do
  not coerce. Recursive value-type declarations are rejected (CORE_V1).
- Records require exactly their declared named fields; declaration order does
  not affect construction. Enum variants have zero or one payload;
  constructors are `E::V` and `E::V(x)` (CORE_V1).
- `+ - * / %` and ordering accept Int; equality accepts identical types
  recursively, including records/enums; logical operators accept Bool. No
  implicit conversions, truthiness, null, float or string concatenation (CORE_V1).
- Lets are immutable; a block's final expression is its result; branch results
  must have the same type. Operation signatures are explicit (CORE_V1).
- Types are interned semantic nodes, not text aliases; nominal identity is the
  declaration, not its name text (SEMANTIC_MODEL).
- Incompatible types use `WOLIO-TYPE-001` (EVIDENCE_MODEL).

## Acceptance criteria
Positive and boundary:
- [ ] Nominal construction and `.value` extraction type-check
- [ ] Records construct with fields in any order; field access types correctly
- [ ] Enum constructors with zero and one payload type-check
- [ ] Recursive structural equality on records/enums type-checks

Rejection and failure:
- [ ] Nominal mismatch, including two nominals over the same representation
- [ ] Missing, extra or duplicate record fields
- [ ] Bad call arity and argument types
- [ ] Recursive value-type declarations
- [ ] Non-Bool conditions, mismatched branches, implicit conversions

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p wolio-cli -- check --project <sem-fixture-project> --json
```
Evidence: `.execution/reports/T021.md` with the SEM fixture list.

## Dependencies and open items
- Depends on: T020; blocks: T022.
- Open: none.

