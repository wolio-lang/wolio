`.blueprint/04-tasks/T305-std-collections-json-modules.md`
# T305: Implement pure `std::collections` and deterministic `std::json`

| Field | Value |
|---|---|
| Milestone | `M13` |
| Priority | `P0` |
| Depends on | T304 |
| Blocks | T306 |
| Acceptance families | V3-STD (`std::collections` and `std::json`) |
| Bounded areas | `crates/wolio-resolver`, `crates/wolio-types`, `crates/wolio-runtime`, `crates/wolio-codegen` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T305.md` |

## Objective
Implement the pure standard library modules `std::collections` (`sort_int`,
`dedup_int`, `reverse`, `zip`, `take`, `drop`) and `std::json` (`Value` enum,
`parse`, canonical sorted-key `stringify` with zero IEEE-754 floating-point
support).

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 2: Standard library `std::*`), [BASELINE-V3.md](../BASELINE-V3.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.2: Industrial standard library)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-STD`, M13 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **`std::collections` types and pure operations (`effects []`):**
  - `record Pair<A, B> { first: A, second: B }`
  - `std::collections::sort_int(list: List<Int>) -> List<Int>` (returns new list in ascending signed 64-bit order).
  - `std::collections::dedup_int(list: List<Int>) -> List<Int>` (sorts and removes duplicate integers, returning strictly increasing unique elements).
  - `std::collections::reverse<T>(list: List<T>) -> List<T>` (returns new list with reversed element order).
  - `std::collections::zip<A, B>(left: List<A>, right: List<B>) -> List<Pair<A, B>>` (truncates to `min(List::len(left), List::len(right))`).
  - `std::collections::take<T>(list: List<T>, count: Int) -> List<T>` (returns empty list if `count <= 0`, full list if `count >= len`).
  - `std::collections::drop<T>(list: List<T>, count: Int) -> List<T>` (returns full list if `count <= 0`, empty list if `count >= len`).
- **`std::json` types and pure operations (`effects []`):**
  - `enum Value { Null, Bool(Bool), Int(Int), Decimal(Decimal), String(String), Array(List<Value>), Object(Map<String, Value>) }`
  - `enum JsonError { SyntaxError(Int), NumberOverflow(Int), DepthLimitExceeded }`
  - `std::json::parse(input: String) -> Result<Value, JsonError>`:
    - Strict RFC 8259 parser with maximum nesting depth `64` (`Err(JsonError::DepthLimitExceeded)` if exceeded).
    - Integer tokens without `.` or exponent parse into `Value::Int(i64)` (`Err(JsonError::NumberOverflow(offset))` on overflow).
    - Fractional tokens `[-]digits.digits` with `<= 6` fractional places parse into exact `Value::Decimal(Decimal)`; scientific notation (`1e5`) or `> 6` fractional places fail with `Err(JsonError::NumberOverflow(offset))` — **never** parsed as `f64`.
    - Duplicate object keys in input overwrite in parse order into canonical `Map<String, Value>`.
  - `std::json::stringify(val: Value) -> String`:
    - Produces deterministic compact JSON with zero whitespace outside strings, RFC 8259 string escaping, exact `Decimal` formatting (trimmed trailing fractional zeros, at least one digit after `.`, e.g., `12.5`), and object keys serialized in canonical UTF-8 byte ascending order (`Map<String, Value>` iteration order).

## Scope
In scope:
- Builtin symbol resolution and generic type checking for `std::collections` and `std::json` in `wolio-resolver` and `wolio-types`.
- Pure runtime evaluation in `wolio-runtime` and C ABI runtime helpers for `std::collections` and `std::json`.
- Marking `std::json::parse` outputs with `UntrustedOrigin` provenance metadata for `T306` validation checks.

Out of scope (owner):
- `Secret<T>` rejection in `std::json` and validation provenance enforcement (`WOLIO-SECURITY-001`, T306).
- `std::crypto` (T307) and capability-gated I/O modules (T308).

## Normative requirements
- `std::collections` and `std::json` require explicit `import` and `dependencies [...]` entries and have empty effect sets (`effects []`) (CORE_V3).
- `std::json` never uses IEEE-754 floats; `std::json::stringify` is a deterministic canonical serializer where two `Value::Object` maps with identical entries inserted in different orders serialize to the exact same JSON string (CORE_V3, IMPLEMENTATION_SPECIFICATION_V3).
- All `std::collections` operations preserve value semantics without mutating input `List<T>` bindings (CORE_V3).

## Acceptance criteria
Positive and boundary:
- [ ] `sort_int`, `dedup_int`, `reverse`, `zip`, `take`, and `drop` return accurate immutable `List` values across empty, singleton, negative-count, and multi-element inputs
- [ ] `std::json::parse` and `std::json::stringify` round-trip nested `Null`, `Bool`, `Int`, `Decimal`, `String`, `Array`, and `Object` values deterministically
- [ ] `std::json::stringify` emits object keys in canonical lexicographical UTF-8 byte order regardless of `Map::insert` order

Rejection and failure:
- [ ] Malformed JSON syntax, nesting depth `> 64`, integer overflow, float exponent notation (`1.5e2`), or `> 6` decimal places return `Result::Err(JsonError::*)` without panicking
- [ ] Calling `std::collections::*` or `std::json::*` without `import` (`WOLIO-RESOLVE-001`) or `dependencies [...]` (`WOLIO-DEPENDENCY-001`) is rejected

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T305.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T305.md` with `V3-STD` (`std::collections`, `std::json`) conformance fixtures.

## Dependencies and open items
- Depends on: T304; blocks: T306.
- Open: none.

