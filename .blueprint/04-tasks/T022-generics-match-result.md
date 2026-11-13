`.blueprint/04-tasks/T022-generics-match-result.md`
# T022: Type builtin generics, exhaustive match and Result handling

| Field | Value |
|---|---|
| Milestone | `M02` |
| Priority | `P0` |
| Depends on | T021 |
| Blocks | T023 |
| Acceptance families | SEM (builtin generics, matches, Result) |
| Bounded areas | types (`crates/wolio-types`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T022.md` |

## Objective
Type `Option<T>`, `Result<T,E>` and `unit()`, enforce exhaustive match, and
enforce the Result-handling obligation, rejecting every other builtin generic.

## Authority
- Normative: [CORE_V1.md](../02-specifications/language/CORE_V1.md) (types and values), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md) (binding and Result-handling clarifications), [SEMANTIC_MODEL.md](../02-specifications/SEMANTIC_MODEL.md) (type application identity)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `SEM`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope: builtin generic applications and constructors, expected-type
resolution of builtin constructors, match typing/exhaustiveness and arm-scoped
payload bindings, the Result-handling check.

Out of scope (owner):
- Declared error sets and Err-variant propagation (T031, which reuses this
  task's Result-handling analysis rather than reimplementing it).
- Execution of match (T024); predicate use of match (T030).

## Normative requirements
- Builtin generics are exactly `Option<T>` and `Result<T,E>`; other builtin
  generics are rejected; `unit()` constructs Unit (CORE_V1).
- Constructors `Option::None`, `Option::Some(x)`, `Result::Ok(x)`,
  `Result::Err(e)`; expected types may resolve builtin constructors; unresolved
  type parameters are errors, not dynamic values (CORE_V1).
- Builtin type applications include their arguments in identity
  (SEMANTIC_MODEL).
- Match is permitted on enums, Option, Result and Bool; one arm per
  constructor; no wildcards or guards; missing and duplicate arms are errors;
  payload bindings exist only within their arm (CORE_V1).
- A Result-valued binding is handled only by exhaustive match or forwarding it
  as a returned expression; assigning, comparing or merely mentioning it does
  not discharge the obligation; ignoring a Result-valued let is an error
  (CORE_V1, INTERFACE_CONTRACTS).
- Match arm errors use `WOLIO-MATCH-001`; type errors `WOLIO-TYPE-001`
  (EVIDENCE_MODEL). For an unhandled Result the registered meaning of
  `WOLIO-ERROR-001` ("undeclared/unhandled error") is the matching code; record
  the mapping used in the report.

## Acceptance criteria
Positive and boundary:
- [ ] Every builtin generic and constructor type-checks, including nested applications
- [ ] Exhaustive matches over enum, Option, Result and Bool type-check
- [ ] A Result handled by exhaustive match or returned directly is accepted

Rejection and failure:
- [ ] Any other builtin generic application, wrong arity of type arguments
- [ ] Unresolvable constructor type parameters
- [ ] Missing and duplicate match arms; payload binding used outside its arm
- [ ] Ignored, compared or merely mentioned Result-valued bindings

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
Evidence: `.execution/reports/T022.md` with the SEM fixture list.

## Dependencies and open items
- Depends on: T021; blocks: T023.
- Open: none.
