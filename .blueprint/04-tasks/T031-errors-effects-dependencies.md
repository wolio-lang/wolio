`.blueprint/04-tasks/T031-errors-effects-dependencies.md`
# T031: Enforce declared errors, transitive effects and operation dependencies

| Field | Value |
|---|---|
| Milestone | `M03` |
| Priority | `P0` |
| Depends on | T030 |
| Blocks | T032 |
| Acceptance families | EFFECT |
| Bounded areas | `crates/wolio-semantic` (effects submodule) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T031.md` |

## Objective
Statically enforce conservative `errors`, transitive `effects` and explicit
`dependencies` clauses across all operations and implementations, completing the
M03 contract clause profile.

## Authority
- Normative: [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md), [CORE_V1.md](../02-specifications/language/CORE_V1.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `EFFECT`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Well-formedness of `errors`, `effects`, `dependencies` and `intent` clauses
  (replacing M02's `WOLIO-FEATURE-001` rejection for M03 contract clauses).
- Static conservative analysis of possible `Result::Err` variants against
  declared `errors`, reusing T022's `Result`-handling analysis.
- Transitive `effects` upper-bound checking and explicit `dependencies` call
  checking, including `console::print`.

Out of scope (owner):
- Predicate typing and runtime guards (T030); `test` and `verify` commands and
  reachable-error test coverage gates (T032); compatibility diffs (T033).

## Normative requirements
- Missing `effects`, `dependencies` or `errors` clauses denote the empty set;
  duplicate list clauses or duplicate entries within a set clause are rejected
  (CORE_V1, MUTATION_SCHEMA_V1).
- Every called operation must be listed in `dependencies`, including local
  calls and `console::print`; type/enum/nominal constructors and `unit()` are
  exempt. Undeclared operation calls fail with `WOLIO-DEPENDENCY-001` (CORE_V1,
  EVIDENCE_MODEL).
- Effects are upper bounds and propagate transitively from every called
  operation. `console::print(String) -> Unit` has effect `Console::Write`. Any
  direct or transitive effect absent from the caller's declared `effects` fails
  with `WOLIO-EFFECT-001` (CORE_V1, CONTRACT_DESIGN, EVIDENCE_MODEL).
- If an operation returns `Result<T,E>`, `E` must be a declared enum and
  `errors` lists its fully qualified variants (`Enum::Variant`). The
  implementation's possible `Err` variants must be a subset of `errors`. A call
  returning `Result<_,E>` conservatively contributes all of the callee's
  declared `errors` unless exhaustive matching handles them. Unused declared
  variants are allowed; for a non-`Result` return type, `errors` must be empty.
  Violations fail with `WOLIO-ERROR-001` (CONTRACT_DESIGN, EVIDENCE_MODEL).

## Acceptance criteria
Positive and boundary:
- [ ] Propagated error and effect sets pass when declared on caller operations
- [ ] `Result` errors handled by exhaustive `match` do not require declaration in caller `errors`
- [ ] Unused declared error variants and `intent` clauses are accepted

Rejection and failure:
- [ ] Undeclared constructed or propagated `Err` variants fail with `WOLIO-ERROR-001`
- [ ] Non-empty `errors` on a non-`Result` operation or non-enum `E` is rejected
- [ ] Direct or hidden transitive effects not declared in `effects` fail with `WOLIO-EFFECT-001`
- [ ] Calling a local, imported or `console::print` operation omitted from `dependencies` fails with `WOLIO-DEPENDENCY-001`
- [ ] Discarded or unhandled `Result` bindings fail with `WOLIO-ERROR-001`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p wolio-cli -- check --project <effect-fixture-project> --json
```
Evidence: `.execution/reports/T031.md` with the EFFECT fixture list.

## Dependencies and open items
- Depends on: T030; blocks: T032.
- Open: none.

