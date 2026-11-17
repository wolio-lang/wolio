`.blueprint/04-tasks/T205-uint-bytes-decimal-scalars.md`
# T205: Implement `UInt`, `Bytes`, exact 128-bit `Decimal` with explicit rounding, and checked bitwise operations

| Field | Value |
|---|---|
| Milestone | `M09` |
| Priority | `P0` |
| Depends on | T204 |
| Blocks | T206, T207, T208 |
| Acceptance families | V2-DATA (scalar types, exact Decimal math, and bitwise operations) |
| Bounded areas | `crates/wolio-semantic` (submodules `resolver`, `types`), `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T205.md` |

## Objective
Implement type checking and deterministic runtime evaluation for V2 scalar
types—64-bit unsigned `UInt`, immutable `Bytes`, exact signed 128-bit
fixed-scale `Decimal` (scale `10^6`) with explicit rounding modes, and checked
bitwise operations for systems programming—with checked arithmetic and zero
IEEE-754 floating-point representations.

## Authority
- Normative: [CORE_V2.md](../02-specifications/language/CORE_V2.md) (Section 1 & Section 3: Builtin types and deterministic operations), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md) (Section 3.1 & 3.2: Exact `Decimal` and checked bitwise ops)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-DATA`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Type representations (`crates/wolio-semantic::types`):**
  - `Type::UInt`: unsigned 64-bit integer (`0`..`18446744073709551615`). Constructed via `UInt::of(Int)` (checked `>= 0` at runtime, faulting with `WOLIO-RUNTIME-001` if `< 0`). Supports `+`, `-`, `*`, `/`, `%`, `==`, `!=`, `<`, `<=`, `>`, `>=` (no unary `-`).
  - `Type::Bytes`: immutable byte sequence. Builtin constructors/ops: `Bytes::empty() -> Bytes`, `Bytes::from_utf8(String) -> Bytes`, `Bytes::len(Bytes) -> Int`, `==`, `!=`.
  - `Type::RoundingMode`: enum `HalfEven`, `HalfUp`, `Floor`, `Ceil`, `Truncate`.
  - `Type::Decimal`: signed 128-bit fixed-point decimal with fixed scale `6` (`10^-6` precision, stored as `i128` where `1.000000` is `1_000_000_i128`).
    - Builtin constructors:
      - `Decimal::of(value: Int) -> Decimal`: computes `i128::from(value) * 1_000_000`.
      - `Decimal::from_scaled(mantissa: Int, scale: Int) -> Decimal`: requires `0 <= scale <= 6`; computes `i128::from(mantissa) * 10_i128.pow(6 - scale as u32)`. If `scale < 0` or `scale > 6`, faults at runtime with `WOLIO-RUNTIME-001` (or `WOLIO-CONTRACT-004` inside a contract predicate).
    - Operators on `Decimal`: unary `-`, binary `+`, `-`, `*`, `/`, `%`, `==`, `!=`, `<`, `<=`, `>`, `>=`.
    - Strict trapping multiplication: `(a: i128, b: i128)` computes exact product `p = i128_wide_mul(a, b)`; if `p % 1_000_000 != 0` or `p / 1_000_000` overflows `i128`, faults with `WOLIO-RUNTIME-001` (`WOLIO-CONTRACT-004` in contracts).
    - Strict trapping division: `(a: i128, b: i128)` faults if `b == 0`, if `(a * 1_000_000)` overflows, or if `(a * 1_000_000) % b != 0` (inexact division is never silently rounded).
    - Explicit rounding operations:
      - `Decimal::mul_round(a: Decimal, b: Decimal, mode: RoundingMode) -> Decimal`
      - `Decimal::div_round(a: Decimal, b: Decimal, mode: RoundingMode) -> Decimal`
      - Div by zero faults with `WOLIO-RUNTIME-001`. Fractional digits beyond scale `10^-6` are resolved strictly by `mode`.
  - **Checked bitwise operations on `Int` and `UInt`:**
    - `bit_and(a: T, b: T) -> T`
    - `bit_or(a: T, b: T) -> T`
    - `bit_xor(a: T, b: T) -> T`
    - `bit_not(a: T) -> T`
    - `bit_shl(a: T, n: Int) -> T`
    - `bit_shr(a: T, n: Int) -> T` (arithmetic shift for `Int`, logical shift for `UInt`)
    - Shift amounts `n` must satisfy `0 <= n < 64`. Shift amounts `< 0` or `>= 64` fault immediately with `WOLIO-RUNTIME-001` (or `WOLIO-CONTRACT-004`).
- **Nominal wrapping:** `pub type Money = Decimal;`, `pub type Count = UInt;`, `pub type Payload = Bytes;` work identically to V1 nominal types (`Money(d)` and `m.value`).

## Scope
In scope:
- Builtin type resolution and type checking for `UInt`, `Bytes`, `Decimal`, `RoundingMode`, and bitwise functions in `wolio-semantic` (submodules `resolver`, `types`).
- Runtime representation and checked evaluation in `wolio-runtime`.
- Contract predicate evaluation support (`WOLIO-CONTRACT-004` on arithmetic/scale/shift fault inside a contract clause).

Out of scope (owner):
- Bounded `for` loop checking and execution (T206); `List<T>` and `Map<K, V>` (T207); `format(...)` (T208); native LLVM lowering for `Decimal`/`UInt`/`Bytes`/bitwise (T213).

## Normative requirements
- `UInt`, `Bytes`, `Decimal`, and bitwise functions are available only when `[language] version = "2"` (or higher); referencing them under `version = "1"` fails with `WOLIO-FEATURE-001` (CORE_V2, EXTENSION_BOUNDARIES).
- `UInt` arithmetic is checked 64-bit unsigned arithmetic; underflow below `0`, overflow above `u64::MAX`, or division/modulo by zero faults with `WOLIO-RUNTIME-001` in bodies or `WOLIO-CONTRACT-004` in contracts (CORE_V2, EVIDENCE_MODEL).
- `Decimal` uses exact 128-bit scaled integer arithmetic (scale `6`, exact to `10^-6`); no binary floating-point (`f32`/`f64`) is used anywhere; standard `*` and `/` fault if remainder is non-zero, while `Decimal::mul_round` and `Decimal::div_round` require explicit `RoundingMode` (CORE_V2, IMPLEMENTATION_SPECIFICATION_V2).
- Bitwise operations require matching types for binary operands and strictly bounded shift amounts (`0 <= n < 64`); out-of-range shifts fault with `WOLIO-RUNTIME-001` (CORE_V2).
- No implicit coercion exists between `Int`, `UInt`, and `Decimal` (`WOLIO-TYPE-001`) (CORE_V2).

## Acceptance criteria
Positive and boundary:
- [ ] `UInt::of`, `Decimal::of`, `Decimal::from_scaled(12345, 2)` (`123.450000`), `Bytes::empty()`, and `Bytes::from_utf8("...")` type-check and evaluate deterministically
- [ ] `Decimal::div_round(Decimal::of(1), Decimal::of(3), RoundingMode::HalfEven)` produces exact `"0.333333"`
- [ ] `Decimal::mul_round` accurately calculates tax/discounts with standard rounding modes without precision fault
- [ ] Bitwise operations (`bit_and`, `bit_or`, `bit_xor`, `bit_not`, `bit_shl`, `bit_shr`) evaluate accurately for both `Int` and `UInt`
- [ ] `UInt` boundary values (`0` and `u64::MAX`) evaluate accurately

Rejection and failure:
- [ ] Mixing `Int`, `UInt`, and `Decimal` in binary operators without explicit conversion fails with `WOLIO-TYPE-001`
- [ ] Unary negation `-u` on `UInt` fails with `WOLIO-TYPE-001`
- [ ] `UInt` underflow (`0 - 1`), `UInt` overflow, `UInt::of(-1)`, `Decimal` overflow, division by zero, invalid `from_scaled` scale (`-1` or `7`), and inexact standard `Decimal` division (`Decimal::of(1) / Decimal::of(3)`) fault with `WOLIO-RUNTIME-001` (or `WOLIO-CONTRACT-004` inside contracts)
- [ ] Bitwise shifts with `n < 0` or `n >= 64` fault with `WOLIO-RUNTIME-001`
- [ ] Mismatched types in `bit_and(int_val, uint_val)` fail with `WOLIO-TYPE-001`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no `f32`/`f64` in codebase; no unrelated changes
- [ ] Report written in `.execution/reports/T205.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T205.md` with `V2-DATA` scalar and `Decimal` boundary fixtures.

## Dependencies and open items
- Depends on: T204; blocks: T206, T207, T208.
- Open: none.

