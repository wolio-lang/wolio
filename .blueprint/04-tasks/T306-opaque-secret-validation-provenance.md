`.blueprint/04-tasks/T306-opaque-secret-validation-provenance.md`
# T306: Enforce opaque `Secret<T>` and validation provenance (`WOLIO-SECURITY-001`)

| Field | Value |
|---|---|
| Milestone | `M13` |
| Priority | `P0` |
| Depends on | T305 |
| Blocks | T307 |
| Acceptance families | V3-SECRET |
| Bounded areas | `crates/wolio-diagnostics`, `crates/wolio-types`, `crates/wolio-effects`, `crates/wolio-analyzer`, `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T306.md` |

## Objective
Implement the opaque `Secret<T>` (`Secret<String>`, `Secret<Bytes>`) type,
compile-time sink and construction restrictions, runtime/journal redaction, and
external-input smart-constructor validation provenance checking (`WOLIO-SECURITY-001`,
Taxonomy `W11`).

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 3: Opaque `Secret<T>` and validation provenance), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md), [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md) (`W11`)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.3: Opaque `Secret<T>` and input validation provenance)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-SECRET`, M13 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Opaque `Secret<T>` typing rules (`crates/wolio-types`):**
  - `Secret<T>` accepts only `T = String` or `T = Bytes` (any other type argument fails with `WOLIO-TYPE-001`).
  - `Secret<T>` has **no** `.value` field, **no** `==` or `!=` equality operator, and **no** ordering operator (`WOLIO-SECURITY-001`).
- **Forbidden sinks and construction (`crates/wolio-effects`, `crates/wolio-analyzer`):**
  1. **No production literal construction (`WOLIO-SECURITY-001`):** Calling `Secret(...)` or `Secret::for_test(...)` inside a production `operation` or `predicate` fails with `WOLIO-SECURITY-001`. Production code may only obtain `Secret<T>` from `std::env::read_secret(key: String) -> Result<Secret<String>, EnvError>` (or `Secret<Bytes>` via `std::secret::to_bytes(Secret<String>) -> Secret<Bytes>`, requiring `effects [Secret::Use]`). `Secret::for_test(val)` is permitted only inside `test` and `property` declarations.
  2. **No leakage to logging, formatting, or serialization sinks (`WOLIO-SECURITY-001`):** Passing `Secret<T>`—or any `record`, `enum`, `Option`, `Result`, `List`, or `Map` transitively containing a `Secret<T>` field—to `console::print`, `format(...)`, `std::json::stringify`, `std::fs::write_text`, or returning `Secret<T>` from `[entrypoint]` fails with `WOLIO-SECURITY-001`.
  3. **Authorized `Secret::Use` sinks:** Only operations declaring `effects [Secret::Use]` (such as `std::crypto::hmac_sha256`) may consume a `Secret<T>` parameter.
- **External input validation provenance (`WOLIO-SECURITY-001`):**
  - Values extracted from external untrusted sources (`std::json::parse`, `std::fs::read_text`, `std::net::http_send`) are tagged with `Provenance::Untrusted` in `wolio-effects`.
  - Wrapping an `Untrusted` value in a domain nominal constructor `DomainType(raw)` inside an operation is rejected with `WOLIO-SECURITY-001` unless that operation's `ensures` clause establishes a validating `predicate` over the returned `DomainType` (e.g., `ensures match (result) { Result::Ok(id) => ValidAccountId(id), Result::Err(_) => true };`).
- **Runtime and diagnostic redaction (`crates/wolio-runtime`, `crates/wolio-diagnostics`):**
  - `Value::SecretString` and `Value::SecretBytes` render as `"<REDACTED:Secret>"` in `Debug`, `Display`, diagnostic violation snapshots, and `.wolio-log/` records, and overwrite their buffer bytes with `0u8` on `Drop`.

## Scope
In scope:
- `Type::Secret` and `WOLIO-SECURITY-001` checks in `wolio-types`, `wolio-effects`, and `wolio-analyzer`.
- `std::env::read_secret` (`effects [Secret::Use]`), `std::secret::to_bytes`, and test-only `Secret::for_test`.
- External input validation provenance tracking (`Provenance::Untrusted` -> validated smart constructor) emitting `WOLIO-SECURITY-001`.
- Redaction of `Secret<T>` in `wolio-runtime` and `wolio-diagnostics`.

Out of scope (owner):
- `std::crypto::hmac_sha256` and `constant_time_eq` (T307); `std::fs` and `std::net` runtime implementations (T308).

## Normative requirements
- Constructing `Secret<T>` from a literal in production code, extracting `.value`, comparing with `==`/`!=`, or passing `Secret<T>` (directly or nested inside a composite type) to `console::print`, `format(...)`, or `std::json` fails at compile time with `WOLIO-SECURITY-001` (CORE_V3, EVIDENCE_MODEL, V3-TASKS).
- External input values from `std::json` (and in T308 `std::fs`/`std::net`) cannot construct domain nominal types unless guarded by a smart constructor operation whose `ensures` verifies a `predicate` over the constructed value (`WOLIO-SECURITY-001`) (CORE_V3, AI_BUG_TAXONOMY `W11`).
- No `Secret<T>` plaintext may ever appear in compiler diagnostics, contract failure messages, or `.wolio-log/` files (EVIDENCE_MODEL, V3-TASKS).

## Acceptance criteria
Positive and boundary:
- [ ] `Secret<String>` and `Secret<Bytes>` pass through authorized `effects [Secret::Use]` operations and `Secret::for_test(...)` in `test` blocks without exposing plaintext
- [ ] External strings parsed via `std::json::parse` pass provenance validation when converted through a smart constructor operation whose `ensures` checks a `predicate`
- [ ] Runtime contract failure on an operation taking `Secret<String>` prints `"<REDACTED:Secret>"` in the diagnostic payload and never leaks the secret characters

Rejection and failure:
- [ ] Literal `Secret("api_key")` or `Secret::for_test("...")` in a production `operation` fails with `WOLIO-SECURITY-001`
- [ ] `secret.value`, `secret == other`, `console::print(secret)`, `format("{}", secret)`, or `std::json::stringify` on a value containing `Secret<T>` fails with `WOLIO-SECURITY-001`
- [ ] Constructing a nominal `AccountId(raw_json_str)` directly from `std::json::parse` output without a validating `predicate` in `ensures` fails with `WOLIO-SECURITY-001`
- [ ] `Secret<Int>` or `Secret<Record>` fails with `WOLIO-TYPE-001`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T306.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T306.md` with `V3-SECRET` secret-leak, redaction, and validation provenance conformance fixtures.

## Dependencies and open items
- Depends on: T305; blocks: T307.
- Open: none.

