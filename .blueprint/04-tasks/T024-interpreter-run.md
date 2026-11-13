`.blueprint/04-tasks/T024-interpreter-run.md`
# T024: Implement the interpreter, runtime and `run` command

| Field | Value |
|---|---|
| Milestone | `M02` |
| Priority | `P0` |
| Depends on | T023 |
| Blocks | T030 |
| Acceptance families | RUN; completes M02 |
| Bounded areas | runtime/interpreter (`crates/wolio-runtime`), CLI (`crates/wolio-cli`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T024.md` |

## Objective
Execute checked `CORE_V1` programs from the manifest entrypoint with checked
64-bit integer arithmetic and captured console output through `wolio run`.

## Authority
- Normative: [CORE_V1.md](../02-specifications/language/CORE_V1.md) (milestone staging rule), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) (interpreter first), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md) (CLI and output), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Decision: [ADR-008](../01-architecture/decisions/ADR-008-multi-version-anti-workaround-roadmap.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `RUN`, M02 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Crate:** `crates/wolio-runtime` (depends on `wolio-ast`, `wolio-semantic`, `wolio-diagnostics`; never depends on LLVM or `wolio-codegen`).
- **M02 vs M03 clause staging rule (`ADR-008`, `CORE_V1.md`):**
  - Declarative `effects [...]`, `dependencies [...]`, `errors [...]`, and `intent "..."` clauses are parsed and **permitted** in M02 programs (so operations calling `console::print` can declare `effects [Console::Write]; dependencies [console::print];`), while static verification of those sets is reported `not_run` until `T031`.
  - Any operation containing `requires` or `ensures` predicate clauses fails `wolio run` at M02 with `WOLIO-FEATURE-001` (exit `1`) until `T030` implements runtime predicate guards.
- **`wolio run --json` envelope:** Includes `"output": "<captured_console_stdout>"` and `"result": { "value": 42 }` (or `"value": null` for `Unit`).

## Scope
In scope: sequential evaluation of typed semantic expressions, checked `Int` arithmetic, immutable values, `console::print`, entrypoint selection, `run` human/JSON output and exit status, and `WOLIO-FEATURE-001` rejection of `requires`/`ensures` clauses at M02.

Out of scope (owner):
- Contract predicate evaluation and guards (T030); static `errors`/`effects`/`dependencies` set checks (T031); `test` (T032); native execution and `build` (T050–T053).

## Normative requirements
- Left-to-right evaluation; `&&`, `||`, `if`, and `match` evaluate only the selected continuation (CORE_V1).
- Overflow, division by zero, and `MIN / -1` are runtime faults (`WOLIO-RUNTIME-001`, exit `1`); division truncates toward zero; remainder takes the dividend's sign; `MIN % -1` is `0`; arithmetic never wraps (CORE_V1, INTERFACE_CONTRACTS).
- Values are immutable with no observable identity; immutable storage may be shared via reference counting (CORE_V1, DELIVERY_POLICY).
- `console::print(String) -> Unit` is the sole host adapter; no clock, randomness, environment, or network (CORE_V1, DELIVERY_POLICY).
- Entry `main() -> Unit` or `main() -> Int` is selected by `[entrypoint]`, must be public; libraries may omit it, but `run` requires it (CORE_V1, INTERFACE_CONTRACTS).
- Human `run` prints console output then the `Int` result and newline; JSON adds `output` and `result.value` (integer, or `null` for `Unit`); exit status follows `EVIDENCE_MODEL` (`0` on normal return, `1` on fault/rejection, `2` on missing entrypoint/config), never the returned `Int` (INTERFACE_CONTRACTS).
- Operations with `requires` or `ensures` clauses produce `WOLIO-FEATURE-001` at M02 instead of executing unchecked (CORE_V1, ADR-008).

## Acceptance criteria
Positive and boundary:
- [ ] `Unit` and `Int` entrypoints execute; `Int` prints as result, not process exit code
- [ ] Selected branches only; short-circuiting (`&&`, `||`, `if`, `match`) is observable through avoided faults/output
- [ ] Checked limits: `i64::MAX`, `i64::MIN`, truncating division, remainder sign, `MIN % -1 == 0`
- [ ] Console output is captured in JSON `output` without corrupting JSON stdout; declarative `effects`/`dependencies`/`errors`/`intent` clauses are accepted in M02 programs

Rejection and failure:
- [ ] Overflow, divide by zero, and `MIN / -1` fault with `WOLIO-RUNTIME-001`
- [ ] Missing or non-public entrypoint fails explicitly
- [ ] Programs containing `requires` or `ensures` fail `wolio run` at M02 with `WOLIO-FEATURE-001`

Completion:
- [ ] All M02 DELIVERY_PLAN checks pass: pure examples execute, bad types/cycles rejected, cache deletion reproduces the graph, boundary arithmetic faults
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p wolio-cli -- run --project <run-fixture-project> --json
```
Evidence: `.execution/reports/T024.md` with RUN fixtures and exit codes.

## Dependencies and open items
- Depends on: T023; blocks: T030.
- Open: none.
