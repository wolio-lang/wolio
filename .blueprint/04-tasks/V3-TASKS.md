`.blueprint/04-tasks/V3-TASKS.md`
# V3 Task Contracts (`M12`–`M15`: `T301`–`T315`)

Normative overview and quick-reference index for [BASELINE-V3.md](../BASELINE-V3.md),
[CORE_V3.md](../02-specifications/language/CORE_V3.md), and
[IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md).
Every V3 task (`T301`–`T315`) has its own complete, standalone self-contained
contract file linked below:

- **M12 (`T301`–`T303`):** [T301-v2-defect-triage-regressions.md](T301-v2-defect-triage-regressions.md), [T302-cow-refcount-collections.md](T302-cow-refcount-collections.md), [T303-v3-version-isolation-lsp-profiling.md](T303-v3-version-isolation-lsp-profiling.md)
- **M13 (`T304`–`T307`):** [T304-std-string-math-modules.md](T304-std-string-math-modules.md), [T305-std-collections-json-modules.md](T305-std-collections-json-modules.md), [T306-opaque-secret-validation-provenance.md](T306-opaque-secret-validation-provenance.md), [T307-std-crypto-module.md](T307-std-crypto-module.md)
- **M14 (`T308`–`T311`):** [T308-capability-gated-time-fs-net.md](T308-capability-gated-time-fs-net.md), [T309-hermetic-test-with-mock-sandbox.md](T309-hermetic-test-with-mock-sandbox.md), [T310-linear-resource-modifies-old-snapshots.md](T310-linear-resource-modifies-old-snapshots.md), [T311-atomic-transaction-rollback.md](T311-atomic-transaction-rollback.md)
- **M15 (`T312`–`T315`):** [T312-property-fuzz-shrinking-verifier.md](T312-property-fuzz-shrinking-verifier.md), [T313-agent-protocol-v3-mutations.md](T313-agent-protocol-v3-mutations.md), [T314-v3-ir-llvm-formatter-lsp-parity.md](T314-v3-ir-llvm-formatter-lsp-parity.md), [T315-v3-industrial-benchmark-release.md](T315-v3-industrial-benchmark-release.md)

These tasks become eligible only after V2 (`M08`–`M11`) completion and owner
acceptance of the V3 execution start.

---

## M12 — V2 Bug-Fix, Memory & Performance Gate (`T301`–`T303`)

### T301: Triage V2 defects and build `BUG-*` V2 regression suite
- **Depends on:** `T216` (and owner V3 gate approval) | **Blocks:** `T302`
- **Bounded areas:** `tests/conformance/regressions/`
- **Acceptance families:** `V3-REGRESSION`
- **Self-contained contract:**
  - Convert all compiler, loop-verifier, protocol v2, formatter, LSP, and native parity defects found during V2 (`M09`–`M11`) into reproducible `BUG-nnnn` conformance fixtures and fix them across interpreter and LLVM backends.

### T302: Implement copy-on-write (CoW) reference-counting optimization for `List<T>` and `Map<K, V>`
- **Depends on:** `T301` | **Blocks:** `T303`
- **Bounded areas:** `crates/wolio-runtime`, `crates/wolio-codegen`
- **Acceptance families:** `V3-REGRESSION`
- **Self-contained contract:**
  - Optimize `List::push` and `Map::insert`/`remove` in `wolio-runtime` (and native ABI) to mutate in place when reference count is `1` while preserving pure immutable value semantics when shared (`rc > 1`), verified by zero-leak and differential parity tests.

### T303: Enforce `[language] version = "1" | "2" | "3"` isolation and large-workspace LSP profiling
- **Depends on:** `T302` | **Blocks:** `T304`
- **Bounded areas:** `crates/wolio-cli`, `crates/wolio-lsp`, `crates/wolio-parser`
- **Acceptance families:** `V3-REGRESSION`
- **Self-contained contract:**
  - Gate all `CORE_V3` syntax behind `[language] version = "3"`, preserving exact V1 and V2 behavior for `version = "1"` and `"2"`.
  - Verify incremental LSP responsiveness and memory bounds on a 100-module synthetic V2 workspace.

---

## M13 — Industrial Standard Library (`std::*`) & Opaque `Secret<T>` (`T304`–`T307`)

### T304: Implement pure `std::string` and `std::math` standard library modules
- **Depends on:** `T303` | **Blocks:** `T305`
- **Bounded areas:** `crates/wolio-resolver`, `crates/wolio-types`, `crates/wolio-runtime`, `crates/wolio-codegen`
- **Acceptance families:** `V3-STD`
- **Self-contained contract:**
  - Expose builtin `std::string` (`len`, `slice`, `contains`, `starts_with`, `ends_with`, `trim`, `split`, `parse_int`, `parse_decimal`) and `std::math` (`abs`, `min`, `max`, `clamp`, `pow_checked`) requiring explicit `import std::string;` and `dependencies [...]` entries.
  - Ensure identical behavior and checked-overflow guarantees in both interpreter and native LLVM backends.

### T305: Implement pure `std::collections` and deterministic `std::json`
- **Depends on:** `T304` | **Blocks:** `T306`
- **Bounded areas:** `crates/wolio-types`, `crates/wolio-runtime`, `crates/wolio-codegen`
- **Acceptance families:** `V3-STD`
- **Self-contained contract:**
  - Implement `std::collections` (`sort_int`, `dedup_int`, `reverse`, `zip`, `take`, `drop`).
  - Implement `std::json` (`Value` enum, `parse(String) -> Result<Value, JsonError>`, `stringify(Value) -> String` with canonical sorted keys and no float support).

### T306: Enforce opaque `Secret<T>` and validation provenance (`WOLIO-SECURITY-001`)
- **Depends on:** `T305` | **Blocks:** `T307`
- **Bounded areas:** `crates/wolio-types`, `crates/wolio-effects`, `crates/wolio-analyzer`
- **Acceptance families:** `V3-SECRET`
- **Self-contained contract:**
  - Implement `Secret<String>` and `Secret<Bytes>` as opaque types without `.value`, equality `==`, or literal construction in production code (`WOLIO-SECURITY-001`).
  - Reject passing `Secret<T>` to `console::print`, `format(...)`, or `std::json` (`WOLIO-SECURITY-001`).
  - Redact any `Secret<T>` bindings from diagnostics, `.wolio-log/` records, and crash dumps.

### T307: Implement `std::crypto` (`sha256`, `hmac_sha256`, `constant_time_eq`)
- **Depends on:** `T306` | **Blocks:** `T308`
- **Bounded areas:** `crates/wolio-runtime`, `crates/wolio-codegen`, `crates/wolio-effects`
- **Acceptance families:** `V3-STD`, `V3-SECRET`
- **Self-contained contract:**
  - Implement `std::crypto::sha256(Bytes) -> Bytes`, `constant_time_eq(Bytes, Bytes) -> Bool`, and `hmac_sha256(Secret<Bytes>, Bytes) -> Bytes` (requiring effect `Secret::Use`).

---

## M14 — Capability-Gated I/O, Deterministic Test Mocks & Transactions (`T308`–`T311`)

### T308: Implement capability-gated `std::time`, `std::fs`, and `std::net`
- **Depends on:** `T307` | **Blocks:** `T309`
- **Bounded areas:** `crates/wolio-effects`, `crates/wolio-runtime`, `crates/wolio-codegen`
- **Acceptance families:** `V3-IO`
- **Self-contained contract:**
  - Implement `std::time::now` (`Clock::Read`), `std::fs::read_text`/`write_text` (`Fs::Read`, `Fs::Write`), and `std::net::http_send` (`Net::Http`).
  - Require every external effect used by a package to be authorized in `wolio.toml` `[capabilities]`; adding a capability requires `HUMAN_REQUIRED` approval.

### T309: Enforce hermetic test sandbox and mandatory `with_mock` handlers (`WOLIO-EFFECT-002`)
- **Depends on:** `T308` | **Blocks:** `T310`
- **Bounded areas:** `crates/wolio-verify`, `crates/wolio-types`, `crates/wolio-runtime`
- **Acceptance families:** `V3-IO`
- **Self-contained contract:**
  - Parse and type-check `with_mock { <effect_op> => <pure_handler_expr>, ... } in { <test_body> }` inside `test` and `property` blocks.
  - Reject any `test` or `property` block that transitively invokes a non-console external effect (`Clock::Read`, `Fs::*`, `Net::*`, `Db::*`) without a covering `with_mock` handler (`WOLIO-EFFECT-002`).

### T310: Implement linear `resource` handles, `modifies [...]` frames, and `old(expr)` snapshots (`WOLIO-RESOURCE-001`)
- **Depends on:** `T309` | **Blocks:** `T311`
- **Bounded areas:** `crates/wolio-parser`, `crates/wolio-types`, `crates/wolio-contracts`, `crates/wolio-runtime`
- **Acceptance families:** `V3-TX`
- **Self-contained contract:**
  - Enforce non-aliasing on `resource <Handle>` parameters: passing the same resource binding twice in one call fails statically with `WOLIO-RESOURCE-001`.
  - Require operations mutating a resource to declare `modifies [handle];` (`WOLIO-RESOURCE-001`).
  - Evaluate `old(expr)` in `ensures` at operation entry before body execution and bind the snapshot value for postcondition evaluation.

### T311: Implement `transaction (handle) { ... }` atomic rollback and postcondition verification
- **Depends on:** `T310` | **Blocks:** `T312`
- **Bounded areas:** `crates/wolio-runtime`, `crates/wolio-codegen`, `crates/wolio-verify`
- **Acceptance families:** `V3-TX`
- **Self-contained contract:**
  - Execute `transaction (handle) { ... }` against transactional resource adapters: on `Result::Err` return or runtime/contract fault, roll back the resource to its entry snapshot before evaluating error-branch `ensures` postconditions.
  - Verify rollback invariants under fault injection before and during commit.

---

## M15 — Contract Property Fuzzing, Agent Protocol v3 & Industrial Release (`T312`–`T315`)

### T312: Implement `property` declarations and `wolio verify --fuzz` shrinking counterexamples (`WOLIO-FUZZ-001`)
- **Depends on:** `T311` | **Blocks:** `T313`
- **Bounded areas:** `crates/wolio-parser`, `crates/wolio-verify`, `crates/wolio-cli`
- **Acceptance families:** `V3-FUZZ`
- **Self-contained contract:**
  - Parse and type-check `property <name> for <op>(<params>) { where <pred>; <bool_expr> }`.
  - Implement deterministic PRNG + boundary value generator (seeded with `0x574F4C494F33`, default 256 valid inputs per operation/property satisfying `requires`/`where`) and deterministic input shrinking.
  - Emit `WOLIO-FUZZ-001` with the minimal shrunk counterexample whenever a generated input violates `ensures` or the `property` body, defeating hardcoded unit-test workarounds.

### T313: Upgrade Agent Protocol to v3 (`property`, `resource`, `with_mock`, `Secret` rules)
- **Depends on:** `T312` | **Blocks:** `T314`
- **Bounded areas:** `crates/wolio-agent`, `crates/wolio-analyzer`
- **Acceptance families:** `V3-FUZZ`, `AGENT`
- **Self-contained contract:**
  - Support `protocol_version: 3` with typed AST payloads for `property`, `resource`, `modifies`, `old`, `transaction`, and `with_mock`.
  - Enforce `HUMAN_REQUIRED` tier on any `modifies` frame, `Secret` sink, or `[capabilities]` change, and require `wolio verify --fuzz` to pass before `apply`/`finalize`.

### T314: Complete V3 Native LLVM, `wolio fmt`, and `wolio lsp` parity
- **Depends on:** `T313` | **Blocks:** `T315`
- **Bounded areas:** `crates/wolio-ir`, `crates/wolio-codegen`, `crates/wolio-format`, `crates/wolio-lsp`
- **Acceptance families:** `V3-RELEASE`, `NATIVE`, `TOOL`
- **Self-contained contract:**
  - Lower all V3 `std::*`, `Secret<T>`, `resource`, and `transaction` constructs to `VerifiedWolioIr` and LLVM with 100% interpreter/native parity.
  - Update `wolio fmt` and `wolio lsp` for all `CORE_V3` syntax.

### T315: Execute full W01–W15 industrial benchmark and package Wolio V3 release
- **Depends on:** `T314` | **Blocks:** none (completes V3)
- **Bounded areas:** `benchmarks/`, release packaging
- **Acceptance families:** `V3-RELEASE`, `BENCH`
- **Self-contained contract:**
  - Execute the expanded paired benchmark covering all 15 `AI_BUG_TAXONOMY.md` failure classes (`W01`–`W15`) across Wolio V3, Go, Rust, and TypeScript.
  - Publish the reproducible 10,000-resample bootstrap report and Linux x86_64 V3 `.tar.gz` distribution archive.

