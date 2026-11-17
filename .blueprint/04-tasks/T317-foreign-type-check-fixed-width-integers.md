`.blueprint/04-tasks/T317-foreign-type-check-fixed-width-integers.md`
# T317: Type-check linear foreign types (`WOLIO-FOREIGN-002`) and fixed-width integer scalar types

| Field | Value |
|---|---|
| Milestone | `M14` |
| Priority | `P0` |
| Depends on | T316 |
| Blocks | T318 |
| Acceptance families | V3-FFI, V3-DATA (foreign types, non-storability, and fixed-width integers) |
| Bounded areas | `crates/wolio-semantic` (submodules `types`, `effects`), `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T317.md` |

## Objective
Implement static type-checking and runtime representation for linear opaque
`foreign type` declarations, statically prohibiting them from being stored in
fields of ordinary `record` types or `enum` variants (`WOLIO-FOREIGN-002`),
implement the six fixed-width integer scalar types (`UInt8`..`UInt32`,
`Int8`..`Int32`) with checked arithmetic and explicit conversions, and enforce
the `Foreign::Call` effect on foreign operation invocations.

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 7: Foreign Function Interface & Section 8: Fixed-width integers), [DIAGNOSTIC_SPEC.md](../02-specifications/verification/DIAGNOSTIC_SPEC.md) (`WOLIO-FOREIGN-002`)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.7 & Section 3.8)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-FFI` and `V3-DATA`, M14 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Linear opaque `foreign type` checking (`crates/wolio-semantic::types`, `crates/wolio-semantic::effects`):**
  - Represents `Type::Foreign(TypeId, LibraryName)`.
  - Foreign types are opaque: field access (`h.field`) or method calls on foreign types are prohibited (`WOLIO-TYPE-001`).
  - Foreign types are linear:
    - Aliasing rebinding (`let h2 = h;`) without consuming `h` is prohibited (`WOLIO-RESOURCE-001`).
    - Passing the same foreign handle twice in one call (`f(h, h)`) is prohibited (`WOLIO-RESOURCE-001`).
  - **Storage restriction (`WOLIO-FOREIGN-002`):**
    - Declaring a field of a `record` with a foreign type, an `enum` variant with a foreign type payload, or instantiating a collection `List<ForeignType>`, `Map<K, ForeignType>`, `Option<ForeignType>`, or `Result<ForeignType, E>` fails compilation with `WOLIO-FOREIGN-002`:
      ```json
      {
        "code": "WOLIO-FOREIGN-002",
        "severity": "error",
        "message": "Foreign type cannot be stored in composite type or collection",
        "violation": {
          "type": "DeviceHandle",
          "container": "record DeviceState"
        }
      }
      ```
  - **Scope-bound auto-drop:** At the exit of its enclosing lexical scope, a live foreign type value triggers an automatic drop call in `wolio-runtime` to the library's destructor glue (`<lib>_<type>_drop(ptr)`), guaranteeing zero memory leaks.
- **Fixed-width integer scalar types (`crates/wolio-semantic::types`, `crates/wolio-runtime`):**
  - Types:
    - `UInt8`: unsigned 8-bit (`0`..`255`)
    - `UInt16`: unsigned 16-bit (`0`..`65535`)
    - `UInt32`: unsigned 32-bit (`0`..`4294967295`)
    - `Int8`: signed 8-bit (`-128`..`127`)
    - `Int16`: signed 16-bit (`-32768`..`32767`)
    - `Int32`: signed 32-bit (`-2147483648`..`2147483647`)
  - **Checked arithmetic:** Operators `+`, `-`, `*`, `/`, `%` perform checked arithmetic within the exact range of the type. Exceeding capacity or division by zero triggers runtime fault `WOLIO-RUNTIME-001` (or `WOLIO-CONTRACT-004` inside contracts). Zero silent truncation or overflow wraparound.
  - **Checked bitwise operations:** `bit_and`, `bit_or`, `bit_xor`, `bit_not`, `bit_shl`, `bit_shr`. Shift amounts `n` must satisfy `0 <= n < width` (`8`, `16`, or `32`), otherwise faulting with `WOLIO-RUNTIME-001`.
  - **Explicit checked conversions:**
    - Narrowing: `UInt8::from_uint(x: UInt) -> UInt8` (faults if `x > 255`), `UInt16::from_uint(x)` (faults if `x > 65535`), `UInt32::from_uint(x)` (faults if `x > 4294967295`), `Int8::from_int(x: Int) -> Int8` (faults if `x < -128 || x > 127`), `Int16::from_int(x)` (faults if `x < -32768 || x > 32767`), `Int32::from_int(x)` (faults if `x < -2147483648 || x > 2147483647`).
    - Widening: `val.to_uint() -> UInt` and `val.to_int() -> Int`.
    - No implicit coercion: passing `UInt8` where `UInt` is expected, or vice versa, fails with `WOLIO-TYPE-001`.
- **Effect checking (`crates/wolio-semantic::effects`):**
  - Invoking a `foreign operation` requires the caller to declare `effects [Foreign::Call]`; omitting it fails with `WOLIO-EFFECT-001`.

## Scope
In scope:
- Type-checking `ForeignTypeDecl` and foreign parameters in `wolio-semantic::types`.
- Enforcing non-storability of foreign types in records, enums, and collections emitting `WOLIO-FOREIGN-002`.
- Adding `Type::{UInt8, UInt16, UInt32, Int8, Int16, Int32}` and `Value::{UInt8, UInt16, UInt32, Int8, Int16, Int32}`.
- Checked arithmetic, bitwise operations, and explicit conversion operations for fixed-width integers in `wolio-runtime`.
- Enforcing `effects [Foreign::Call]` on foreign calls.

Out of scope (owner):
- Foreign operation contract pre/post checking and error code marshalling (T318).
- LLVM IR lowering of foreign bindings and fixed-width native types (T319).

## Normative requirements
- Attempting to store a `foreign type` in a `record` field, an `enum` payload, or any collection must fail static checking with `WOLIO-FOREIGN-002` (CORE_V3, DIAGNOSTIC_SPEC).
- Fixed-width integer arithmetic must fault with `WOLIO-RUNTIME-001` on overflow, underflow, or divide by zero (CORE_V3).
- Conversions between integers of different widths must be explicit (`UInt8::from_uint`, `.to_uint()`); implicit coercion is rejected with `WOLIO-TYPE-001` (CORE_V3).

## Acceptance criteria
Positive and boundary:
- [ ] Passing a `foreign type` to a `foreign operation` as a linear argument succeeds
- [ ] `UInt8::from_uint(255)` evaluates to `UInt8(255)` and `UInt8(200) + UInt8(55)` evaluates to `UInt8(255)`
- [ ] `Int8::from_int(-128)` evaluates to `Int8(-128)` and `Int8(-128) - Int8(1)` triggers runtime fault `WOLIO-RUNTIME-001`
- [ ] Explicit `.to_uint()` and `.to_int()` conversions widen without loss

Rejection and failure:
- [ ] Declaring `record Container { dev: DeviceHandle }` fails with `WOLIO-FOREIGN-002`
- [ ] Declaring `enum DevOption { Some(DeviceHandle), None }` fails with `WOLIO-FOREIGN-002`
- [ ] Passing `UInt8` directly to a parameter expecting `UInt` without explicit conversion fails with `WOLIO-TYPE-001`
- [ ] `UInt8::from_uint(256)` triggers runtime fault `WOLIO-RUNTIME-001`
- [ ] Calling a foreign operation without declaring `effects [Foreign::Call]` fails with `WOLIO-EFFECT-001`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T317.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T317.md` with `V3-FFI` and `V3-DATA` type checking and overflow fault conformance fixtures.

## Dependencies and open items
- Depends on: T316; blocks: T318.
- Open: none.

