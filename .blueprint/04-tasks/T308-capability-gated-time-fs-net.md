`.blueprint/04-tasks/T308-capability-gated-time-fs-net.md`
# T308: Implement capability-gated `std::time`, `std::fs`, and `std::net`

| Field | Value |
|---|---|
| Milestone | `M14` |
| Priority | `P0` |
| Depends on | T307 |
| Blocks | T309 |
| Acceptance families | V3-IO (`std::time`, `std::fs`, `std::net`, and `[capabilities]`) |
| Bounded areas | `crates/wolio-resolver`, `crates/wolio-semantic`, `crates/wolio-types`, `crates/wolio-effects`, `crates/wolio-runtime`, `crates/wolio-codegen` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T308.md` |

## Objective
Implement the capability-gated standard library modules `std::time`, `std::fs`,
and `std::net` and enforce package-level `[capabilities]` effect authorization
in `wolio.toml` (`WOLIO-EFFECT-001` and `HUMAN_REQUIRED` approval tier).

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 2 & Section 4: Capability-gated external effects), [APPROVAL_MATRIX_V1.md](../02-specifications/protocol/APPROVAL_MATRIX_V1.md), [BASELINE-V3.md](../BASELINE-V3.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.2 & Section 3.4)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-IO`, M14 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **`wolio.toml` `[capabilities]` policy (`crates/wolio-semantic`, `crates/wolio-effects`):**
  ```toml
  [capabilities]
  allowed_effects = ["Console::Write", "Clock::Read", "Fs::Read", "Fs::Write", "Net::Http"]
  ```
  - Recognized effect identifiers in V3: `"Console::Write"`, `"Clock::Read"`, `"Fs::Read"`, `"Fs::Write"`, `"Net::Http"`, `"Db::Query"`, `"Db::Tx"`, `"Secret::Use"`.
  - If `[capabilities]` is omitted in `wolio.toml`, `allowed_effects` defaults to `["Console::Write"]` (preserving 100% compatibility with V1/V2 console programs).
  - If any operation in the workspace declares an effect in `effects [...]` that is not present in `[capabilities].allowed_effects`, `wolio check` fails with `WOLIO-EFFECT-001` (`violation: { "observed": "<Effect>", "allowed_capabilities": [...] }`).
  - Normalized `[capabilities].allowed_effects` is hashed into the project policy snapshot hash (`wolio-semantic`), and any addition to `[capabilities]` requires `HUMAN_REQUIRED` approval.
- **`std::time` types and operations:**
  - `type Timestamp = Int;` (UTC Unix epoch milliseconds), `type Duration = Int;` (milliseconds).
  - `std::time::now() -> Timestamp` (`effects [Clock::Read]`).
  - `std::time::add(ts: Timestamp, delta: Duration) -> Timestamp` and `std::time::diff(a: Timestamp, b: Timestamp) -> Duration` (pure checked arithmetic, `effects []`).
- **`std::fs` types and operations:**
  - `type Path = String;`
  - `enum FsError { NotFound, PermissionDenied, InvalidPath, IoFault(String) }`
  - `std::fs::read_text(path: Path) -> Result<String, FsError>` (`effects [Fs::Read]`, tags returned `String` with `Provenance::Untrusted` for `T306` validation checks).
  - `std::fs::write_text(path: Path, content: String) -> Result<Unit, FsError>` (`effects [Fs::Write]`).
- **`std::net` types and operations:**
  - `record HttpRequest { method: String, url: String, body: String }`
  - `record HttpResponse { status: Int, body: String }`
  - `enum NetError { InvalidUrl, Timeout, ConnectionFailed, ProtocolError(String) }`
  - `std::net::http_send(req: HttpRequest) -> Result<HttpResponse, NetError>` (`effects [Net::Http]`, tags `HttpResponse.body` with `Provenance::Untrusted`).

## Scope
In scope:
- Parsing and hashing `[capabilities]` in `wolio.toml` and checking operation `effects [...]` against `allowed_effects` in `wolio-effects`.
- Resolving, type-checking, and executing `std::time`, `std::fs`, and `std::net` in `wolio-resolver`, `wolio-types`, and `wolio-runtime` (with C ABI runtime hooks for native codegen).
- Propagating `Provenance::Untrusted` on `std::fs::read_text` and `std::net::http_send` outputs.

Out of scope (owner):
- Mandatory `with_mock` test sandbox (`WOLIO-EFFECT-002`, T309); linear `resource` and `transaction` blocks (T310, T311).

## Normative requirements
- Every external effect (`Clock::Read`, `Fs::Read`, `Fs::Write`, `Net::Http`, `Db::Query`, `Db::Tx`, `Secret::Use`) must be both declared in the calling operation's `effects [...]` (propagating transitively) and authorized in `wolio.toml` `[capabilities]` (`WOLIO-EFFECT-001` if missing from either) (CORE_V3, V3-TASKS).
- Modifying `[capabilities]` in `wolio.toml` alters the project policy snapshot hash and requires `HUMAN_REQUIRED` approval (CORE_V3, APPROVAL_MATRIX_V1).
- Strings returned from `std::fs::read_text` and `std::net::http_send` are untrusted external inputs and subject to `T306` validation provenance (`WOLIO-SECURITY-001`) before constructing validated domain nominal types (CORE_V3).

## Acceptance criteria
Positive and boundary:
- [ ] Operations calling `std::time::now`, `std::fs::read_text`, `std::fs::write_text`, and `std::net::http_send` pass `wolio check` when declared in `dependencies [...]`, `effects [...]`, and `wolio.toml` `[capabilities]`
- [ ] Pure `std::time::add` and `std::time::diff` require no effects and fault with `WOLIO-RUNTIME-001` on 64-bit overflow
- [ ] `std::fs::read_text` and `std::fs::write_text` execute deterministically on controlled local temporary paths during `wolio run`

Rejection and failure:
- [ ] Calling `std::time::now`, `std::fs::*`, or `std::net::*` without declaring the effect in `effects [...]` fails with `WOLIO-EFFECT-001`
- [ ] Declaring `effects [Fs::Write]` or `effects [Net::Http]` when omitted from `wolio.toml` `[capabilities]` fails with `WOLIO-EFFECT-001`
- [ ] Passing unvalidated `std::fs::read_text` or `std::net::http_send` output directly into a domain nominal constructor without a validating `predicate` in `ensures` fails with `WOLIO-SECURITY-001`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T308.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T308.md` with `V3-IO` capability-gating and `std::time`/`fs`/`net` conformance results.

## Dependencies and open items
- Depends on: T307; blocks: T309.
- Open: none.

