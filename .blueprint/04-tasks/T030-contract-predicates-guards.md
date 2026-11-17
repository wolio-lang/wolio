`.blueprint/04-tasks/T030-contract-predicates-guards.md`
# T030: Check contract predicates and enforce runtime guards

| Field | Value |
|---|---|
| Milestone | `M03` |
| Priority | `P0` |
| Depends on | T024 |
| Blocks | T031 |
| Acceptance families | CONTRACT |
| Bounded areas | `crates/wolio-semantic` (contracts submodule), runtime (`crates/wolio-runtime`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T030.md` |

## Objective
Validate `requires` and `ensures` predicates statically and evaluate them in
source order as runtime guards around every operation call.

## Authority
- Normative: [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md), [CORE_V1.md](../02-specifications/language/CORE_V1.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `CONTRACT`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Static well-formedness and `Bool` typing of `requires` and `ensures` clauses.
- Runtime guard evaluation order for `requires` and `ensures`, including nested
  operation calls and `Result` postconditions.
- Structured contract violation and predicate-fault diagnostics.

Out of scope (owner):
- Static `errors`, `effects` and `dependencies` set enforcement (T031).
- `test` and `verify` CLI commands and evidence aggregation (T032).
- Contract compatibility and `inspect`/`impact` queries (T033).
- Native IR/LLVM contract guard lowering (T050–T053).

## Normative requirements
- Repeated `requires` and `ensures` clauses are conjunctions evaluated in source
  order (CONTRACT_DESIGN).
- `requires` sees operation parameters; `ensures` additionally sees `result`
  bound to the entire return value, including `Result<T,E>`; `result` in
  `requires` is rejected with `WOLIO-CONTRACT-001` (CONTRACT_DESIGN,
  EVIDENCE_MODEL).
- Every predicate must have type `Bool`. Predicates permit literals, immutable
  parameter/`result` fields, builtin and nominal constructors, operators, `if`
  and exhaustive `match`. User operation calls in predicates are forbidden even
  if pure (`WOLIO-CONTRACT-001`); literal `false` in `requires` is diagnosed
  statically (CONTRACT_DESIGN, EVIDENCE_MODEL).
- Call arguments evaluate before callee `requires`; `requires` runs before the
  operation body; `ensures` runs after the body result and before returning to
  the caller (CONTRACT_DESIGN).
- Failed `requires` is a caller violation (`WOLIO-CONTRACT-002`); failed
  `ensures` is an implementation violation (`WOLIO-CONTRACT-003`); arithmetic
  faults during predicate evaluation produce `WOLIO-CONTRACT-004`, never a
  passing check (CONTRACT_DESIGN, EVIDENCE_MODEL).
- Contract violations abort the invocation without rolling back effects already
  performed. Diagnostics identify clause, operation, caller and source span
  without dumping runtime argument values (CONTRACT_DESIGN, EVIDENCE_MODEL).

## Acceptance criteria
Positive and boundary:
- [ ] Multiple `requires` and `ensures` clauses evaluate as conjunctions in source order
- [ ] Nominal inputs, record/enum fields and `Result` match postconditions execute with guards
- [ ] Nested operation calls evaluate caller/callee `requires` and `ensures` in exact boundary order

Rejection and failure:
- [ ] `result` in `requires`, non-`Bool` predicates and operation calls in predicates fail with `WOLIO-CONTRACT-001`
- [ ] Literal `false` in `requires` is diagnosed statically; runtime false `requires` fails with `WOLIO-CONTRACT-002`
- [ ] False `ensures` fails with `WOLIO-CONTRACT-003`
- [ ] Overflow, division by zero or `MIN / -1` inside a predicate fails with `WOLIO-CONTRACT-004`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p wolio-cli -- check --project <contract-fixture-project> --json
cargo run -p wolio-cli -- run --project <contract-fixture-project> --json
```
Evidence: `.execution/reports/T030.md` with the CONTRACT fixture list.

## Dependencies and open items
- Depends on: T024; blocks: T031.
- Open: none.

