`.blueprint/04-tasks/T304-std-string-math-modules.md`
# T304: Implement pure `std::string` and `std::math` standard library modules

| Field | Value |
|---|---|
| Milestone | `M13` |
| Priority | `P0` |
| Depends on | T303 |
| Blocks | T305 |
| Acceptance families | V3-STD (`std::string` and `std::math`) |
| Bounded areas | `crates/wolio-semantic` (submodules `resolver`, `types`, `effects`), `crates/wolio-runtime`, `crates/wolio-codegen` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T304.md` |

## Objective
Implement the builtin pure standard library modules `std::string` and
`std::math` under `[language] version = "3"`, requiring explicit
`import std::string;` / `import std::math;` and `dependencies [...]`
declarations and guaranteeing identical checked behavior in both the
interpreter and native LLVM backend.

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 2: Standard library `std::*`), [BASELINE-V3.md](../BASELINE-V3.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.2: Industrial standard library)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-STD`, M13 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Explicit import & dependency rule (`crates/wolio-semantic::resolver`, `crates/wolio-semantic::effects`):**
  - Builtin `std::*` modules are never implicitly in scope. A module must declare `import std::string;` or `import std::math;` (failing with `WOLIO-RESOLVE-001` if referenced without import, or `WOLIO-FEATURE-001` if imported under `version = "1"` or `"2"`).
  - User modules cannot declare `module std` (`std` is a reserved keyword).
  - Every `operation` calling a `std::string::*` or `std::math::*` operation must list it in `dependencies [...]` (e.g., `dependencies [std::string::trim, std::math::clamp];`), or fail with `WOLIO-DEPENDENCY-001` (and unused entries fail with `WOLIO-ANALYZER-003`).
  - Pure `std::string::len`, `contains`, `starts_with`, `ends_with`, and `std::math::min`, `max` may also be called inside `predicate`, `requires`, `ensures`, `bound`, and `invariant` clauses.
- **`std::string` types and operations (`effects []`):**
  - `enum StringError { InvalidInt, OverflowInt, InvalidDecimal, OverflowDecimal }`
  - `std::string::len(s: String) -> Int` (returns UTF-8 byte length `>= 0`).
  - `std::string::slice(s: String, start: Int, end: Int) -> Option<String>` (returns `Option::Some(sub)` iff `0 <= start <= end <= len(s)` and both `start` and `end` lie on valid UTF-8 codepoint boundaries; otherwise returns `Option::None` — never panics).
  - `std::string::contains(s: String, needle: String) -> Bool`
  - `std::string::starts_with(s: String, prefix: String) -> Bool`
  - `std::string::ends_with(s: String, suffix: String) -> Bool`
  - `std::string::trim(s: String) -> String` (strips leading and trailing ASCII/Unicode whitespace).
  - `std::string::split(s: String, sep: String) -> List<String>` (if `sep == ""`, returns single-element `[s]`).
  - `std::string::parse_int(s: String) -> Result<Int, StringError>` (parses optional `-` followed by ASCII digits in `i64::MIN..=i64::MAX`; returns `Err(StringError::InvalidInt)` on non-digits/whitespace or `Err(StringError::OverflowInt)` on range overflow).
  - `std::string::parse_decimal(s: String) -> Result<Decimal, StringError>` (parses exact fixed-point decimal with up to 6 fractional digits into scaled `i128`; more than 6 fractional digits or non-decimal syntax returns `Err(StringError::InvalidDecimal)`; range overflow returns `Err(StringError::OverflowDecimal)`).
- **`std::math` types and operations (`effects []`):**
  - `enum MathError { Overflow }`
  - `std::math::abs(x: Int) -> Int` (faults with `WOLIO-RUNTIME-001` if `x == i64::MIN`).
  - `std::math::min(a: Int, b: Int) -> Int`
  - `std::math::max(a: Int, b: Int) -> Int`
  - `std::math::clamp(x: Int, lo: Int, hi: Int) -> Int` (`requires lo <= hi;` — faults with `WOLIO-CONTRACT-002` if `lo > hi`).
  - `std::math::pow_checked(base: Int, exp: UInt) -> Result<Int, MathError>` (returns `Result::Ok(base^exp)` with `0^0 == 1`, or `Result::Err(MathError::Overflow)` if `i64::checked_pow` overflows).

## Scope
In scope:
- Builtin module registration and `sym_` identities for `std::string` and `std::math` in `wolio-resolver`, `wolio-semantic`, and `wolio-types`.
- Dependency and analyzer constant-folding support for `std::string` and `std::math`.
- Interpreter implementation in `wolio-runtime` and C runtime ABI stubs for native parity (`wolio_rt_std_string_*`, `wolio_rt_std_math_*`).

Out of scope (owner):
- `std::collections` and `std::json` (T305); `Secret<T>` (T306); `std::crypto` (T307); `std::time`/`fs`/`net` (T308).

## Normative requirements
- Calling any `std::string` or `std::math` operation without `import std::<module>;` fails with `WOLIO-RESOLVE-001`, and calling it from an `operation` without declaring it in `dependencies [...]` fails with `WOLIO-DEPENDENCY-001` (CORE_V3, IMPLEMENTATION_SPECIFICATION_V3).
- `std::string::slice` never panics on negative, out-of-bounds, or mid-codepoint indices, returning `Option::None` instead (IMPLEMENTATION_SPECIFICATION_V3).
- `std::math::abs(i64::MIN)` faults with `WOLIO-RUNTIME-001`, `std::math::clamp(x, lo, hi)` with `lo > hi` faults with `WOLIO-CONTRACT-002`, and `std::math::pow_checked` returns `Result::Err(MathError::Overflow)` on 64-bit signed overflow (IMPLEMENTATION_SPECIFICATION_V3).

## Acceptance criteria
Positive and boundary:
- [ ] All `std::string` (`len`, `slice`, `contains`, `starts_with`, `ends_with`, `trim`, `split`, `parse_int`, `parse_decimal`) and `std::math` (`abs`, `min`, `max`, `clamp`, `pow_checked`) operations type-check and execute deterministically
- [ ] `std::string::slice` returns `Option::None` on negative, reversed (`start > end`), out-of-range, or mid-UTF-8-character byte offsets, and `Option::Some` on valid boundaries
- [ ] `std::string::parse_int` and `parse_decimal` handle boundary values (`-9223372036854775808`, `9223372036854775807`, `0.000001`) and return typed `Result::Err` on invalid strings or >6 fractional places

Rejection and failure:
- [ ] Referencing `std::string::*` without `import std::string;` fails with `WOLIO-RESOLVE-001`
- [ ] Calling `std::string::*` or `std::math::*` in an `operation` without listing it in `dependencies [...]` fails with `WOLIO-DEPENDENCY-001`
- [ ] `std::math::abs(-9223372036854775808)` aborts with `WOLIO-RUNTIME-001` and `std::math::clamp(5, 10, 2)` aborts with `WOLIO-CONTRACT-002`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T304.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T304.md` with `V3-STD` (`std::string`, `std::math`) conformance fixture results.

## Dependencies and open items
- Depends on: T303; blocks: T305.
- Open: none.

