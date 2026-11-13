`.blueprint/04-tasks/T206-bounded-for-loop-guards.md`
# T206: Type-check and execute total bounded `for` loops with `bound` and `invariant` guards

| Field | Value |
|---|---|
| Milestone | `M09` |
| Priority | `P0` |
| Depends on | T205 |
| Blocks | T207 |
| Acceptance families | V2-LOOP |
| Bounded areas | `crates/wolio-resolver`, `crates/wolio-types`, `crates/wolio-contracts`, `crates/wolio-verify`, `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T206.md` |

## Objective
Enforce static binding, typing, and purity rules on V2 `for (item in source with acc: Type = init)`
loops and execute them in the interpreter with mandatory runtime `bound`
(`WOLIO-CONTRACT-005`) and entry/step/exit `invariant` (`WOLIO-CONTRACT-006`)
guards.

## Authority
- Normative: [CORE_V2.md](../02-specifications/language/CORE_V2.md) (Section 4: Total bounded loop semantics), [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md) (Section 3.3: Total bounded loop execution)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-LOOP`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Static resolution & binding rules (`crates/wolio-resolver`):**
  - Neither `item` nor `acc` may be `"result"` or shadow any visible parameter, outer `let` binding, or outer loop variable (`WOLIO-RESOLVE-001`), and `item != acc` (`WOLIO-RESOLVE-001`).
  - Scope of `acc`: visible inside `invariant`, loop `let` statements, and loop trailing body expression; **not** visible inside `iter_source`, `acc_init`, or `bound`.
  - Scope of `item`: visible inside loop `let` statements and loop trailing body expression; **not** visible inside `iter_source`, `acc_init`, `bound`, or `invariant` (since `invariant` is also evaluated on entry before iteration `0` and on exit after the final iteration).
- **Static type & contract rules (`crates/wolio-types`, `crates/wolio-contracts`):**
  - `iter_source`: either `start..end` where both `start` and `end` have type `Int` (producing `item: Int`), or an expression of type `List<T>` (producing `item: T`, connected in `T207`). Any other `iter_source` type fails with `WOLIO-TYPE-001`.
  - `acc_init` and the loop body's final expression must both have type `acc_type` (`WOLIO-TYPE-001`).
  - `bound` and `invariant` must have type `Bool` and obey contract purity rules: no `operation` calls (`WOLIO-CONTRACT-001`).
  - For a range `start..end`, `bound` must include an explicit comparison expression (`<=` or `<`) bounding the range span or endpoint; literal `false` in `bound` or `invariant` fails statically with `WOLIO-CONTRACT-001`.
  - Loops are forbidden inside contract clauses (`requires`, `ensures`, `bound`, `invariant`) and `predicate` bodies (`WOLIO-CONTRACT-001`).
- **Runtime execution order (`crates/wolio-runtime`):**
  1. Evaluate `iter_source` once (for `start..end`, if `start >= end`, the iteration sequence is empty `[]`; otherwise `[start, start + 1, ..., end - 1]`).
  2. Evaluate `acc_init` once to `acc`.
  3. Evaluate `bound` once:
     - If an arithmetic fault occurs in `bound`, abort with `WOLIO-CONTRACT-004`.
     - If `bound` evaluates to `false`, abort with `WOLIO-CONTRACT-005`.
  4. Evaluate **entry** `invariant` with `acc = init`:
     - If an arithmetic fault occurs in `invariant`, abort with `WOLIO-CONTRACT-004`.
     - If `false`, abort with `WOLIO-CONTRACT-006`.
  5. For each element `item` in sequence:
     - Evaluate loop `let` bindings and body expression to `next_acc`.
     - Update `acc = next_acc`.
     - Evaluate **step** `invariant` with updated `acc`:
       - Fault -> `WOLIO-CONTRACT-004`; `false` -> `WOLIO-CONTRACT-006`.
  6. Evaluate **exit** `invariant` with final `acc` (fault -> `WOLIO-CONTRACT-004`; `false` -> `WOLIO-CONTRACT-006`) and yield `acc`.

## Scope
In scope:
- Resolution, non-shadowing, type checking, and contract purity validation for `ForExpr` and `start..end` ranges.
- Interpreter execution of bounded `for` loops with `bound` (`WOLIO-CONTRACT-005`) and entry/step/exit `invariant` (`WOLIO-CONTRACT-006`) guards.
- Recording `runtime_guard` obligations for loop `bound` and `invariant` clauses in `wolio-verify`.

Out of scope (owner):
- `List<T>` construction and iteration (T207); calling `predicate` declarations in `bound`/`invariant` (T209); tautological `invariant true;` analyzer rule (T212); LLVM lowering of loops (T213).

## Normative requirements
- `iter_source` and `acc_init` evaluate once before iteration begins; `start > end` or `start == end` yields `0` iterations while still checking `bound` and entry/exit `invariant` (CORE_V2).
- `bound` is checked once before iteration `0` and aborts with `WOLIO-CONTRACT-005` if `false` (CORE_V2, EVIDENCE_MODEL).
- `invariant` is checked on entry (`acc = init`), after each iteration step (`acc = next_acc`), and on exit, aborting with `WOLIO-CONTRACT-006` if `false` (CORE_V2, EVIDENCE_MODEL).
- Arithmetic faults inside `bound` or `invariant` emit `WOLIO-CONTRACT-004`; arithmetic faults inside `iter_source`, `acc_init`, or the loop body emit `WOLIO-RUNTIME-001` (CONTRACT_DESIGN, EVIDENCE_MODEL).

## Acceptance criteria
Positive and boundary:
- [ ] Bounded `for (i in 0..n with acc: Int = 0)` loops compute accurate accumulator results
- [ ] Empty ranges (`5..5` and `10..2`) execute `0` iterations, check `bound` and entry/exit `invariant`, and return `acc_init`
- [ ] Nested bounded `for` loops with distinct variable names execute deterministically

Rejection and failure:
- [ ] Shadowing outer bindings with `item` or `acc`, or referencing `item` inside `bound`/`invariant`, fails with `WOLIO-RESOLVE-001`
- [ ] Non-`Bool` `bound`/`invariant` or calling an `operation` inside `bound`/`invariant` fails with `WOLIO-CONTRACT-001`
- [ ] Accumulator body type mismatch fails with `WOLIO-TYPE-001`
- [ ] Runtime `bound` evaluating to `false` aborts with `WOLIO-CONTRACT-005`
- [ ] Runtime `invariant` evaluating to `false` on entry (`acc = init`), mid-loop step, or final iteration aborts with `WOLIO-CONTRACT-006`
- [ ] Overflow or division by zero inside `bound`/`invariant` aborts with `WOLIO-CONTRACT-004`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T206.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T206.md` with `V2-LOOP` positive and negative conformance fixtures.

## Dependencies and open items
- Depends on: T205; blocks: T207.
- Open: none.

