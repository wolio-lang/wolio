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

## 5. Stateful resource handles and `transaction` blocks

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

- Resource handles (`resource <Type>`) are linear/non-aliasing parameters: passing
  the same resource handle twice in one call (`f(db, db)`) is rejected statically
  (`WOLIO-RESOURCE-001`).
- Operations mutating a resource must declare `modifies [handle];` and effect
  `Db::Tx` (or resource effect).
- `old(expr)` in `ensures` captures the entry snapshot of a pure inspector over the
  declared resource before the body executes.
- `transaction (handle) { ... }` executes a block against a transactional resource
  adapter: if the block returns `Result::Err(...)` or faults, the adapter rolls back
  the resource state to its entry snapshot and verifies the error-branch `ensures`
  postcondition (`AccountBalance(db, from) == old(...)`).

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

