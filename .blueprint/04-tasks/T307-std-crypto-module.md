`.blueprint/04-tasks/T307-std-crypto-module.md`
# T307: Implement `std::crypto` (`sha256`, `hmac_sha256`, `constant_time_eq`)

| Field | Value |
|---|---|
| Milestone | `M13` |
| Priority | `P0` |
| Depends on | T306 |
| Blocks | T308 (M14 entry) |
| Acceptance families | V3-STD, V3-SECRET; completes M13 |
| Bounded areas | `crates/wolio-resolver`, `crates/wolio-types`, `crates/wolio-effects`, `crates/wolio-runtime`, `crates/wolio-codegen` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T307.md` |

## Objective
Implement `std::crypto` (`sha256`, `hmac_sha256`, `constant_time_eq`) as the
authorized cryptographic sink for `Secret<Bytes>` and `Bytes`, completing
milestone M13 (`V3-STD` and `V3-SECRET`).

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 2 & Section 3), [BASELINE-V3.md](../BASELINE-V3.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.2 & Section 3.3)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-STD` and `V3-SECRET`, M13 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **`std::crypto` operations (`crates/wolio-types`, `crates/wolio-effects`, `crates/wolio-runtime`):**
  1. `std::crypto::sha256(data: Bytes) -> Bytes`:
     - Pure (`effects []`).
     - Computes FIPS 180-4 SHA-256 digest and returns a 32-byte `Bytes` value (`Bytes::len(result) == 32`).
  2. `std::crypto::hmac_sha256(key: Secret<Bytes>, message: Bytes) -> Bytes`:
     - Effectful (`effects [Secret::Use]`).
     - Computes RFC 2104 / FIPS 198-1 HMAC-SHA-256 over `message` using the opaque `key` and returns a 32-byte `Bytes` MAC tag (`Bytes::len(result) == 32`), zeroizing intermediate key-pad buffers before returning.
     - Calling `std::crypto::hmac_sha256` from an operation that omits `Secret::Use` from `effects [...]` fails with `WOLIO-EFFECT-001`.
  3. `std::crypto::constant_time_eq(a: Bytes, b: Bytes) -> Bool`:
     - Pure (`effects []`).
     - If `Bytes::len(a) != Bytes::len(b)`, returns `false` without out-of-bounds reads; when lengths match, compares all bytes using bitwise XOR accumulation (`fold(0u8, |acc, (x, y)| acc | (x ^ y)) == 0`) without data-dependent short-circuit branches.
- **Additional `Bytes` hex helpers (`std::crypto::to_hex(Bytes) -> String`, `std::crypto::from_hex(String) -> Result<Bytes, CryptoError>`):**
  - Pure (`effects []`), lowercase hexadecimal encoding/decoding so digests and test vectors can be inspected deterministically without exposing `Secret<Bytes>`.

## Scope
In scope:
- `std::crypto` module resolution, typing, `dependencies [...]` checking, and `Secret::Use` effect checking in `wolio-resolver`, `wolio-types`, and `wolio-effects`.
- Deterministic SHA-256, HMAC-SHA-256, constant-time equality, and hex encoding/decoding in `wolio-runtime` and C ABI runtime helpers (`wolio_rt_std_crypto_*`).

Out of scope (owner):
- Capability-gated external I/O modules `std::time`, `std::fs`, `std::net` (T308).

## Normative requirements
- `std::crypto::hmac_sha256` accepts `Secret<Bytes>` as its key parameter and requires the caller to declare `effects [Secret::Use]` (`WOLIO-EFFECT-001` if omitted) and `dependencies [std::crypto::hmac_sha256]` (`WOLIO-DEPENDENCY-001` if omitted) (CORE_V3, V3-TASKS).
- Passing a plain `Bytes` instead of `Secret<Bytes>` as the HMAC key fails type checking with `WOLIO-TYPE-001`, preventing hardcoded unredacted HMAC keys in production code (CORE_V3).
- `std::crypto::sha256` and `std::crypto::hmac_sha256` must match official NIST FIPS 180-4 and RFC 4231 test vectors bit-for-bit in both interpreter and native execution (IMPLEMENTATION_SPECIFICATION_V3).

## Acceptance criteria
Positive and boundary:
- [ ] `std::crypto::sha256` matches NIST SHA-256 vectors (empty `Bytes`, `"abc"`, multi-block inputs)
- [ ] `std::crypto::hmac_sha256` with `Secret<Bytes>` matches RFC 4231 HMAC-SHA-256 test vectors and requires `effects [Secret::Use]`
- [ ] `std::crypto::constant_time_eq` returns `true` for identical byte sequences and `false` for differing bytes or lengths

Rejection and failure:
- [ ] Calling `std::crypto::hmac_sha256` without `effects [Secret::Use]` fails with `WOLIO-EFFECT-001`
- [ ] Passing a plain `Bytes` or `String` as the first argument to `hmac_sha256` fails with `WOLIO-TYPE-001`
- [ ] Constructing `Secret<Bytes>` from a literal inside a production operation to feed `hmac_sha256` fails with `WOLIO-SECURITY-001`

Completion:
- [ ] All M13 `DELIVERY_PLAN` and `ACCEPTANCE_MATRIX` (`V3-STD`, `V3-SECRET`) gates pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T307.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T307.md` with NIST/RFC cryptographic vector results and M13 milestone gate summary.

## Dependencies and open items
- Depends on: T306; blocks: T308 (and therefore M14).
- Open: none.

