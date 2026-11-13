`.blueprint/04-tasks/T208-deterministic-typechecked-format.md`
# T208: Implement type-checked deterministic `format(template, ...)`

| Field | Value |
|---|---|
| Milestone | `M09` |
| Priority | `P0` |
| Depends on | T207 |
| Blocks | T209 (M10 entry) |
| Acceptance families | V2-DATA (`format`); completes M09 |
| Bounded areas | `crates/wolio-types`, `crates/wolio-contracts`, `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T208.md` |

## Objective
Implement compile-time template validation and deterministic runtime rendering
for `format(template: String, args...) -> String`, completing M09.

## Authority
- Normative: [CORE_V2.md](../02-specifications/language/CORE_V2.md) (Section 3: `format`), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-DATA`, M09 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Template syntax & compile-time validation (`crates/wolio-types`):**
  - First argument to `format(template, ...)` must be a compile-time string literal (`Expr::StringLiteral`); passing a variable or non-string expression as the template fails with `WOLIO-TYPE-001`.
  - Placeholder syntax in `template`:
    - `{}` is a positional argument placeholder.
    - `{{` and `}}` are escaped literal braces `{` and `}`.
    - Any unclosed `{`, lone `}`, or non-empty format specifier `{:...}` / `{0}` fails with `WOLIO-TYPE-001`.
  - The number of unescaped `{}` placeholders must exactly equal `args.len()`; mismatch fails with `WOLIO-TYPE-001`.
  - Every argument in `args` must have type `Int`, `UInt`, `Decimal`, `Bool`, or `String`.
  - **Strict nominal and composite rejection:** Passing a nominal wrapper (`OrderId`) without `.value`, or passing `Bytes`, `Unit`, `Option`, `Result`, `List`, `Map`, a `record`, or an `enum` fails with `WOLIO-TYPE-001`.
- **Canonical runtime formatting (`crates/wolio-runtime`):**
  - `Int`: base-10 signed ASCII digits (e.g., `-42`, `0`, `100`).
  - `UInt`: base-10 unsigned ASCII digits (e.g., `0`, `18446744073709551615`).
  - `Decimal`: exact fixed 6-decimal-place ASCII string `[-]D+.DDDDDD` (e.g., `Decimal::of(15)` -> `"15.000000"`, `Decimal::from_scaled(-125, 2)` -> `"-1.250000"`, `Decimal::from_scaled(5, 6)` -> `"0.000005"`).
  - `Bool`: `"true"` or `"false"`.
  - `String`: exact UTF-8 string contents.
  - `format(...)` is pure (zero effects) and permitted in both implementation bodies and contract predicates.

## Scope
In scope:
- Compile-time template parsing, placeholder counting, and scalar argument type checking in `wolio-types`.
- Pure runtime evaluation of `format(...)` in `wolio-runtime` and contract predicates (`wolio-contracts`).
- Completing all M09 (`V2-LOOP` and `V2-DATA`) conformance suites.

Out of scope (owner):
- `WOLIO-ANALYZER-004` constant-folding over `format(...)` (T212); LLVM native lowering of `format(...)` (T213); V3 `Secret<T>` rejection (`WOLIO-SECURITY-001`, T306).

## Normative requirements
- `format(template, args...)` requires a string literal template whose `{}` count equals the number of supplied arguments (`WOLIO-TYPE-001`) (CORE_V2).
- Only `Int`, `UInt`, `Decimal`, `Bool`, and `String` are accepted as `format` arguments; unextracted nominal wrappers (without `.value`) and composite types are rejected with `WOLIO-TYPE-001` (CORE_V2).
- `Decimal` values always render deterministically with 6 fractional digits (`[-]D+.DDDDDD`) (IMPLEMENTATION_SPECIFICATION_V2).

## Acceptance criteria
Positive and boundary:
- [ ] `format("order={} qty={} total={} active={}", id.value, qty, total, active)` produces deterministic output for `String`, `Int`, `UInt`, `Decimal`, and `Bool`
- [ ] Zero-placeholder `format("static text")` and escaped braces `format("{{ok}}={}", 1)` evaluate accurately
- [ ] `format(...)` is accepted inside pure expressions and `ensures` postconditions

Rejection and failure:
- [ ] Non-literal template first argument (`format(msg, 1)`) fails with `WOLIO-TYPE-001`
- [ ] Placeholder count mismatch (`format("{} {}", 1)` or `format("{}", 1, 2)`) or malformed braces (`format("{", 1)`) fails with `WOLIO-TYPE-001`
- [ ] Passing a raw nominal type (`format("{}", user_id)` instead of `user_id.value`), record, enum, `Bytes`, `List`, or `Map` fails with `WOLIO-TYPE-001`

Completion:
- [ ] All M09 `DELIVERY_PLAN` and `ACCEPTANCE_MATRIX` (`V2-LOOP` and `V2-DATA`) gates pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T208.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T208.md` with `V2-DATA` `format` fixtures and full M09 milestone verification summary.

## Dependencies and open items
- Depends on: T207; blocks: T209 (and therefore M10).
- Open: none.

