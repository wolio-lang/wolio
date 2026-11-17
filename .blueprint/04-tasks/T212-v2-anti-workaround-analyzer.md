`.blueprint/04-tasks/T212-v2-anti-workaround-analyzer.md`
# T212: Extend anti-workaround analyzer for V2 loops, predicates and constants

| Field | Value |
|---|---|
| Milestone | `M10` |
| Priority | `P0` |
| Depends on | T211 |
| Blocks | T213 (M11 entry) |
| Acceptance families | V2-AGENT, ANALYZER; completes M10 |
| Bounded areas | `crates/wolio-semantic` (submodule `analyzer`), `crates/wolio-verify` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T212.md` |

## Objective
Harden `crates/wolio-semantic::analyzer` for `CORE_V2` so AI agents cannot bypass bounded
loop verification with tautological invariants (`invariant true;`), hide
forbidden constants inside `Decimal`/`UInt`/`format(...)` expressions, or
declare vacuous `predicate` bodies, completing M10.

## Authority
- Normative: [DIAGNOSTIC_SPEC.md](../02-specifications/verification/DIAGNOSTIC_SPEC.md), [CORE_V2.md](../02-specifications/language/CORE_V2.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Decision: [ADR-008](../01-architecture/decisions/ADR-008-multi-version-anti-workaround-roadmap.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-AGENT` and `ANALYZER`, M10 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **V2 Analyzer Extensions (`crates/wolio-semantic::analyzer`):**
  1. **`WOLIO-ANALYZER-001` (Tautological Loop Invariant / Predicate Bypass — `error`):**
     - Reject any `for` loop in a production `implementation` whose `invariant` expression is literal `true`, constant-folds to `true` (e.g., `1 == 1`), or does not reference the accumulator binding `acc` (directly or as an argument to a `predicate` call). This prevents an AI agent from writing `invariant true;` merely to satisfy the parser syntax requirement.
     - Reject any `predicate` declaration whose body is literal `true` or constant-folds to a constant `Bool` without referencing all of its declared parameters.
  2. **`WOLIO-ANALYZER-004` (Extended Constant-Folding Forbidden Literals — `error`):**
     - Extend the constant folder to evaluate pure `UInt` (`UInt::of(...)`, `+ - * / %`), `Decimal` (`Decimal::of(...)`, `Decimal::from_scaled(...)`, `+ - * / %`), and `format("...", <constant_args...>)` expressions.
     - If a folded `UInt` or integer-valued `Decimal` matches `forbidden_int_literals`, or a folded `format(...)` result matches `forbidden_string_literals` in `wolio.toml` `[analysis]`, emit `WOLIO-ANALYZER-004`.
  3. **`WOLIO-ANALYZER-005` (Extended Test-Literal Mirroring — `warning`):**
     - Extend test-literal mirroring detection to `UInt` and `Decimal` test inputs (detecting when a production `if`/`match` compares a parameter directly against a non-boundary `Decimal`/`UInt` constant copied from a `test` block).

## Scope
In scope:
- Tautological loop `invariant` and vacuous `predicate` detection under `WOLIO-ANALYZER-001`.
- Constant folding for `UInt`, `Decimal`, and `format(...)` under `WOLIO-ANALYZER-004`.
- Test-literal mirroring detection for `UInt` and `Decimal` under `WOLIO-ANALYZER-005`.
- Integration with `wolio verify` and `wolio agent` (`plan` / `apply` / `finalize`).

Out of scope (owner):
- Native IR/LLVM lowering for V2 constructs (T213).

## Normative requirements
- Writing `invariant true;` (or an invariant that constant-folds to `true` or ignores `acc`) in a production `for` loop is a verification bypass and fails with `WOLIO-ANALYZER-001` (`error`), blocking `verify`, `apply`, and `finalize` (V2-TASKS, IMPLEMENTATION_SPECIFICATION_V2, DIAGNOSTIC_SPEC).
- Constant-folded `UInt`, `Decimal`, and `format(...)` expressions in production implementations that yield configured `forbidden_int_literals` or `forbidden_string_literals` fail with `WOLIO-ANALYZER-004` (`error`) (V2-TASKS, DIAGNOSTIC_SPEC).
- `WOLIO-ANALYZER-005` warnings on `UInt`/`Decimal` test-literal mirroring are reported in `diagnostics` without failing `verify` or escalating approval tiers (DIAGNOSTIC_SPEC).

## Acceptance criteria
Positive and boundary:
- [ ] Bounded `for` loops with genuine invariants referencing `acc` (or calling a `predicate(acc, ...)` that inspects all parameters) pass analyzer checks
- [ ] `Decimal`, `UInt`, and `format(...)` expressions not matching forbidden literals pass `WOLIO-ANALYZER-004`

Rejection and failure:
- [ ] `invariant true;`, `invariant 1 == 1;`, or an `invariant` that never references `acc` fails with `WOLIO-ANALYZER-001`
- [ ] A `predicate` body that is `true` or ignores one of its parameters fails with `WOLIO-ANALYZER-001`
- [ ] `Decimal::of(99) + Decimal::of(1)` when `100` is in `forbidden_int_literals`, or `format("secret_{}", "key")` when `"secret_key"` is in `forbidden_string_literals`, fails with `WOLIO-ANALYZER-004`
- [ ] Production branch comparing a `Decimal` parameter to an exact non-boundary test literal emits a `WOLIO-ANALYZER-005` warning

Completion:
- [ ] All M10 `DELIVERY_PLAN` and `ACCEPTANCE_MATRIX` (`V2-PRED`, `V2-ARCH`, `V2-AGENT`, `ANALYZER`) gates pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T212.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T212.md` with V2 analyzer fixtures and M10 milestone completion check.

## Dependencies and open items
- Depends on: T211; blocks: T213 (and therefore M11).
- Open: none.

