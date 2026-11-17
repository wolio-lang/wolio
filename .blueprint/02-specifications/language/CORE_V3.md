`.blueprint/02-specifications/language/CORE_V3.md`
# Core language V3 — Standard library, capability I/O, secrets, transactions, and contract fuzzing

Normative for the V3 profile in `BASELINE-V3.md`. Grammar version `3` (`[language] version = "3"`).
`CORE_V3` is a strict, upward-compatible superset of `CORE_V1.md` and `CORE_V2.md`.
Using V3 constructs in a project declaring `[language] version = "1"` or `"2"`
produces `WOLIO-FEATURE-001`.

## 1. Anti-workaround design rationale for V3

In large-scale industrial systems, AI agents introduce high-severity defects across
four boundaries:
1. **Test-hardcoding ("overfitting to unit tests"):** AI writes `if (input == 42) { ... }`
   to pass the 2 unit tests in the file instead of implementing the general algorithm.
2. **Uncontrolled side effects in business logic and tests:** AI calls live clocks,
   random generators, filesystems, or HTTP endpoints inside domain operations or tests.
3. **Secret leakage and unvalidated input trust:** AI logs API keys/passwords in
   diagnostics or passes unvalidated strings directly to database/network sinks.
4. **Partial state corruption on failure:** AI mutates external state across multiple
   calls without atomic rollback boundaries.

V3 closes all four vectors at the language and verification level.

## 2. Standard library (`std::*`)

V3 ships deterministic builtin standard library modules (requiring explicit
`import std::<module>;` and `dependencies [...]` declarations):

| Module | Provided types and operations | Effects |
|---|---|---|
| `std::string` | `len`, `slice`, `contains`, `starts_with`, `ends_with`, `trim`, `split`, `parse_int`, `parse_decimal` | pure (`[]`) |
| `std::collections` | `sort_int`, `dedup_int`, `reverse`, `zip`, ` take`, `drop` over `List<T>` / `Map<K,V>` | pure (`[]`) |
| `std::math` | `abs`, `min`, `max`, `clamp`, `pow_checked` | pure (`[]`) |
| `std::json` | `Value` enum, `parse(String) -> Result<Value, JsonError>`, `stringify(Value) -> String` | pure (`[]`) |
| `std::time` | `Timestamp`, `Duration`, `now() -> Timestamp` | `Clock::Read` for `now` |
| `std::crypto` | `sha256(Bytes) -> Bytes`, `hmac_sha256(Secret<Bytes>, Bytes) -> Bytes`, `constant_time_eq` | `Secret::Use` for HMAC |
| `std::fs` | `read_text(Path) -> Result<String, FsError>`, `write_text(Path, String) -> Result<Unit, FsError>` | `Fs::Read`, `Fs::Write` |
| `std::net` | `http_send(HttpRequest) -> Result<HttpResponse, NetError>` | `Net::Http` |

## 3. Opaque `Secret<T>` and validation provenance

- `Secret<T>` (`T` in `String`, `Bytes`) is an opaque builtin generic wrapper.
- **No literal construction in production source:** `Secret` values can only be
  obtained from capability-gated host adapters (`std::env::read_secret`) or test
  fixtures; constructing a `Secret` from a literal in production code fails with
  `WOLIO-SECURITY-001`.
- **No `.value`, equality `==`, `console::print`, `format(...)`, or `std::json`
  serialization:** `Secret<T>` has no `.value` field and is rejected by `format`,
  `console::print`, and `std::json::stringify` (`WOLIO-SECURITY-001`). Only
  explicitly authorized sinks declaring effect `Secret::Use` may consume `Secret<T>`.
- **Validation provenance:** Domain types produced from external inputs (`std::json`,
  `std::fs`, `std::net`) must pass through a smart constructor operation whose
  `ensures` establishes a `predicate` before entering operations that require the
  validated nominal type.

## 4. Capability-gated external effects and mandatory deterministic test mocks

- Every external effect (`Console::Write`, `Clock::Read`, `Fs::Read`, `Fs::Write`,
  `Net::Http`, `Db::Query`, `Db::Tx`, `Secret::Use`) is both:
  1. Statically tracked in `effects [...]` (propagating transitively as in V1), and
  2. Checked against `[capabilities]` in `wolio.toml` (modifications require
     `HUMAN_REQUIRED` approval).
- **Zero live I/O in `wolio test` and `wolio verify`:** Test blocks (`test` and
  `property`) execute in a hermetic deterministic sandbox. Any operation with
  non-console external effects (`Clock::Read`, `Fs::*`, `Net::*`, `Db::*`) must be
  invoked inside tests using a deterministic `with_mock { ... }` handler table;
  unmocked external effects in tests fail with `WOLIO-EFFECT-002`.

## 5. Stateful resource handles, instantiation, and `transaction` blocks

```wol
operation Transfer(db: resource DbHandle, from: AccountId, to: AccountId, amount: Decimal) -> Result<Unit, TxError> {
    requires ValidTransfer(from, to, amount);
    modifies [db];
    ensures match (result) {
        Result::Ok(_) => AccountBalance(db, from) == old(AccountBalance(db, from)) - amount,
        Result::Err(_) => AccountBalance(db, from) == old(AccountBalance(db, from))
    };
    effects [Db::Tx];
    dependencies [Debit, Credit];
    errors [TxError::InsufficientFunds, TxError::AccountMissing];
}
```

- **Scope-bound lexical acquisition (deadlock resolution):**
  - Resource handles cannot be constructed from raw values or literals in normal code.
  - In production entrypoints and host adapters, a resource is acquired via explicit scope-bound acquisition:
    ```wol
    let db = Resource::acquire::<DbHandle>(config);
    ```
    `Resource::acquire::<T>(...)` is permitted only in package `main` or authorized capability adapter operations.
  - In test fixtures and test blocks, a mock resource handle is instantiated exclusively inside a `with_mock` block:
    ```wol
    with_mock {
        // mock handlers...
    } in {
        let mock_db = Resource::mock::<DbHandle>();
        Transfer(mock_db, from_acc, to_acc, amount)
    }
    ```
- **Strict linearity and non-aliasing rules (`WOLIO-RESOURCE-001`):**
  - **No aliased rebinding:** `let b2 = b;` is forbidden statically. Moving or rebinding a live resource without consuming or transferring the original identifier fails with `WOLIO-RESOURCE-001`.
  - **No duplicate argument passing:** Passing the same resource binding in two or more argument positions of a single call (e.g., `f(db, db)`) fails with `WOLIO-RESOURCE-001`.
  - **No escaping or embedding:** A resource handle cannot be stored in fields of a `record`, `enum` payload, `Option`, `Result`, `List`, or `Map` (`WOLIO-RESOURCE-001`).
  - **No escaping returns:** Returning a resource handle from an operation without explicit linear transfer capability fails with `WOLIO-RESOURCE-001`.
- **Frame conditions and snapshots:**
  - Operations mutating a resource must declare `modifies [handle];` and effect `Db::Tx` (or resource effect). Mutating without `modifies [handle];` fails with `WOLIO-RESOURCE-001`.
  - `old(expr)` in `ensures` captures the entry snapshot of a pure inspector over the declared resource before the body executes.
  - `transaction (handle) { ... }` executes a block against a transactional resource adapter: if the block returns `Result::Err(...)` or faults, the adapter rolls back the resource state to its entry snapshot and verifies the error-branch `ensures` postcondition (`AccountBalance(db, from) == old(...)`).

## 6. Contract-driven property fuzzing (`property` and `wolio verify --fuzz`)

```wol
property clamp_postcondition for Clamp(x: Int) {
    where x >= 0 && x <= 1000000;
    Clamp(x) <= 10 && Clamp(x) >= 0
}
```

- `property <name> for <op>(<params>) { where <precondition>; <bool_expr> }`
  declares a deterministic, seed-reproducible property test.
- `wolio verify` in V3 automatically synthesizes deterministic boundary and PRNG
  inputs (default 256 cases per operation/property using fixed seed `0x574F4C494F33`)
  satisfying `requires` / `where` and verifies both `ensures` and the property body.
- This defeats AI test-hardcoding: an implementation that only handles the 2–3
  literal values present in `test` blocks will fail generated boundary/fuzz inputs
  with `WOLIO-CONTRACT-003` and a minimal shrinking counterexample.

## 7. Foreign Function Interface (Capability-Gated Boundary)

Wolio provides a capability-gated Foreign Function Interface (FFI) designed specifically for safe systems programming and AI vibe coding: AI agents and application developers are strictly prevented from authoring raw pointers, manual memory allocators, or unverified `unsafe` blocks.

```wol
pub foreign type DeviceHandle from "libhardware";

pub foreign operation ReadRegister(
    dev: DeviceHandle,
    offset: UInt16
) -> Result<UInt8, ForeignError> from "libhardware" {
    requires offset < 1024;
    ensures match (result) {
        Result::Ok(val) => val <= 255,
        Result::Err(_) => true
    };
    effects [Foreign::Call];
    errors [ForeignError::NullPointer, ForeignError::IoFailure, ForeignError::InvalidResponse];
}
```

- **Linear, opaque foreign types (`foreign type`):**
  - Declared as `foreign type <Type> from "<library>";`.
  - Opaque: internal memory layout and pointers cannot be accessed or inspected in Wolio code.
  - Linear: cannot be duplicated, copied, or aliased.
  - **Storage restriction (`WOLIO-FOREIGN-002`):** A foreign type is strictly forbidden from being stored in fields of ordinary `record` types, `enum` payloads, `List`, `Map`, `Option`, or `Result`.
  - **Scope-bound auto-drop:** A foreign type binding automatically invokes compiler-generated destructor glue (e.g., `<library>_<type>_drop(handle)`) at the end of its enclosing lexical scope, guaranteeing zero resource or memory leaks.
- **Contract-guarded foreign operations (`foreign operation`):**
  - Declared with pre-conditions (`requires`), post-conditions (`ensures`), effects (`effects [Foreign::Call]`), and typed errors (`errors [...]`).
  - Compiler generates deterministic C glue code. The C glue validates `requires` pre-conditions before crossing the C ABI boundary, translates parameters, calls the foreign function, marshals C return codes into `Result<T, ForeignError>`, and verifies `ensures` postconditions upon return.
  - Developers and AI agents never write C headers, pointer casts, or `unsafe` blocks.
- **Capability gating via `wolio.toml` (`WOLIO-FOREIGN-001`):**
  - Every external library referenced in `from "<library>"` must be explicitly registered in `wolio.toml`:
    ```toml
    [foreign]
    allowed_libraries = ["libhardware", "libsqlite3"]
    ```
  - Attempting to declare or import a `foreign type` or `foreign operation` from an unlisted library produces compile-time diagnostic `WOLIO-FOREIGN-001`.
  - Any edit to `[foreign].allowed_libraries` in `wolio.toml` is classified as `HUMAN_REQUIRED` in the agent protocol and approval matrix (`APPROVAL_MATRIX_V1.md`).
- **Mandatory `Foreign::Call` effect and hermetic test mocking (`WOLIO-EFFECT-002`):**
  - Any operation invoking a foreign operation must declare `effects [Foreign::Call]` in its contract.
  - Live foreign calls are strictly forbidden in `wolio test` and `wolio verify`. Any test executing code with effect `Foreign::Call` must supply a mock implementation in a `with_mock` block; unmocked foreign calls in tests fail with `WOLIO-EFFECT-002`.

## 8. Fixed-width integer scalar types for interop & hardware registers

To support hardware register manipulation, network protocols, binary serialisation, and foreign function interop without introducing raw pointers or C undefined behavior, V3 introduces six exact fixed-width integer scalar types:

| Type | Bit width | Signedness | Min value | Max value | Native LLVM IR Type |
|---|---|---|---|---|---|
| `UInt8` | 8-bit | Unsigned | `0` | `255` | `i8` |
| `UInt16` | 16-bit | Unsigned | `0` | `65535` | `i16` |
| `UInt32` | 32-bit | Unsigned | `0` | `4294967295` | `i32` |
| `Int8` | 8-bit | Signed | `-128` | `127` | `i8` |
| `Int16` | 16-bit | Signed | `-32768` | `32767` | `i16` |
| `Int32` | 32-bit | Signed | `-2147483648` | `2147483647` | `i32` |

- **Checked arithmetic with zero silent truncation:**
  - Standard arithmetic operators `+`, `-`, `*`, `/`, `%` perform checked arithmetic within the exact range of the type.
  - Overflow, underflow, or division by zero immediately triggers a runtime fault (`WOLIO-RUNTIME-001` or `WOLIO-CONTRACT-004` within contracts).
  - Silent wraparound and truncation are completely eliminated by design.
- **Checked bitwise operations:**
  - Functional bitwise primitives: `bit_and`, `bit_or`, `bit_xor`, `bit_not`, `bit_shl`, `bit_shr`.
  - Shift amounts `n` are checked: `0 <= n < 8` for `UInt8`/`Int8`, `0 <= n < 16` for `UInt16`/`Int16`, and `0 <= n < 32` for `UInt32`/`Int32`. Shift amounts outside these bounds fault with `WOLIO-RUNTIME-001`.
- **Explicit checked conversions:**
  - Explicit narrowing constructors validate limits at runtime:
    - `UInt8::from_uint(x: UInt) -> UInt8` (faults with `WOLIO-RUNTIME-001` if `x > 255`)
    - `UInt16::from_uint(x: UInt) -> UInt16` (faults if `x > 65535`)
    - `UInt32::from_uint(x: UInt) -> UInt32` (faults if `x > 4294967295`)
    - `Int8::from_int(x: Int) -> Int8` (faults if `x < -128 || x > 127`)
    - `Int16::from_int(x: Int) -> Int16` (faults if `x < -32768 || x > 32767`)
    - `Int32::from_int(x: Int) -> Int32` (faults if `x < -2147483648 || x > 2147483647`)
  - Explicit widening methods:
    - `UInt8::to_uint(val: UInt8) -> UInt`
    - `UInt16::to_uint(val: UInt16) -> UInt`
    - `UInt32::to_uint(val: UInt32) -> UInt`
    - `Int8::to_int(val: Int8) -> Int`
    - `Int16::to_int(val: Int16) -> Int`
    - `Int32::to_int(val: Int32) -> Int`
  - Zero implicit widening or narrowing coercions: passing `UInt8` where `UInt` is expected, or vice versa, fails type-checking with `WOLIO-TYPE-001`.


