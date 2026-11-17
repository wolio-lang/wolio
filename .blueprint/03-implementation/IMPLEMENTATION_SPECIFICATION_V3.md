`.blueprint/03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md`
# Wolio implementation specification V3 (`M12`–`M15`)

Status: revision 1.5 freeze candidate. This document is the authoritative
crate-level implementation specification for Wolio V3 (`[language] version = "3"`,
milestones `M12`–`M15`, tasks `T301`–`T319`). It extends
[IMPLEMENTATION_SPECIFICATION_V1.md](IMPLEMENTATION_SPECIFICATION_V1.md) and
[IMPLEMENTATION_SPECIFICATION_V2.md](IMPLEMENTATION_SPECIFICATION_V2.md) without
introducing new Cargo crates or breaking V1 (`[language] version = "1"`) or V2
(`[language] version = "2"`) isolation.

## 1. V3 compiler and tooling pipeline extensions

```text
source (.wol) + wolio.toml ([language] version = "1" | "2" | "3", [architecture], [capabilities], [foreign])
  -> wolio-lexer (promotes V3 reserved tokens secret/Secret/resource/transaction/modifies/old/property/std/foreign/with_mock/where/fn)
  -> wolio-parser / wolio-ast (parses PropertyDecl, ForeignTypeDecl, ForeignOpDecl, ResourceParam, ModifiesClause, OldExpr, TransactionExpr, WithMockExpr, FixedWidthInts, std::* paths; gates V3 AST behind language version "3")
  -> wolio-semantic (unified compiler middle-end across submodules):
       ├── resolver: resolves builtin std::* namespaces, foreign types/ops, property/resource scopes; validates [foreign] allowed_libraries
       ├── types: type-checks Secret<T>, linear resource handles, linear opaque foreign types (non-storable in records/enums WOLIO-FOREIGN-002), fixed-width integers (UInt8..UInt32, Int8..Int32 with checked conversions), std::* builtin signatures, WithMockExpr handler signatures, TransactionExpr Result<T,E> types
       ├── graph: computes sym_ IDs with kinds "property", "foreign_type", "foreign_op", hashes [capabilities] & [foreign] policies into snapshot hash, tracks FFI dependency and modifies edges
       ├── contracts: validates modifies [handle], evaluates old(expr) purity over modified resources, type-checks property where/body predicates, enforces contract boundary on foreign operations
       ├── effects: enforces [capabilities] and [foreign] allowed_libraries authorization (WOLIO-FOREIGN-001), checks Secret<T> sink & validation provenance (WOLIO-SECURITY-001), enforces mandatory with_mock in tests for non-console effects and Foreign::Call (WOLIO-EFFECT-002), enforces linear resource non-aliasing & modifies completeness (WOLIO-RESOURCE-001)
       └── analyzer: rejects hardcoded secret constructions, vacuous property clauses, foreign storage violations, and unvalidated external-to-domain flows
  -> wolio-verify (executes V2 BUG-* regressions, runs hermetic with_mock test sandbox rejecting unmocked live I/O & Foreign::Call with WOLIO-EFFECT-002, executes deterministic contract/property fuzzer & shrinker seeded with 0x574F4C494F33 emitting WOLIO-FUZZ-001)
  -> wolio-runtime (executes Copy-on-Write refcounted List<T>/Map<K,V>, std::* operations, redacted Secret<T>, Resource::acquire / Resource::mock handles, foreign C glue dispatch tables, old(expr) snapshots, and atomic transaction rollback adapters)
  -> wolio-ir (lowers CoW last-use collection ops, fixed-width scalars to i8/i16/i32, std::* intrinsics, foreign extern call regions, old(expr) entry snapshots, and structured TransactionRegion rollback blocks to VerifiedWolioIr)
  -> wolio-codegen (lowers V3 VerifiedWolioIr to verified textual LLVM IR .ll with native i8/i16/i32 mappings, emits extern C FFI bindings, links with clang and runtime C ABI)
```

Tooling extensions in V3:
- `wolio-agent`: adds `protocol_version: 3` (`wolio.mutation.v3`) supporting typed AST payloads for `property`, `resource` parameters, `modifies` clauses, `old(expr)`, `transaction (handle) { ... }`, `with_mock { ... } in { ... }`, `foreign type`, `foreign operation`, and fixed-width integer scalar types, enforcing `HUMAN_REQUIRED` approval on `[capabilities]`, `[foreign]`, `modifies`, or `Secret<T>` sink changes and requiring `wolio verify --fuzz` to pass before `apply`/`finalize`.
- `wolio-format`: formats `property` declarations, `foreign type` and `foreign operation` declarations, fixed-width integer types, `modifies [...]` clauses, `old(...)` expressions, `transaction (handle) { ... }` blocks, and `with_mock { ... } in { ... }` expressions canonically while verifying `sym_` ID preservation.
- `wolio-lsp`: adds semantic tokens, hover, go-to-definition, find-references, and workspace rename for `std::*` modules, `foreign` types and operations, fixed-width integer types, `property` declarations, `resource` parameters, `with_mock` handlers, and `old(...)` expressions, profiled for `< 50 ms` incremental responsiveness on 100-module workspaces.

## 2. Crate-by-crate V3 implementation responsibilities (`T301`–`T319`)

No new Cargo crates are created in V3; all V3 functionality extends the consolidated
14-crate DAG established in V1 (`IMPLEMENTATION_SPECIFICATION_V1.md`) and V2
(`IMPLEMENTATION_SPECIFICATION_V2.md`):

| Crate | V3 Tasks | V3 Data Structures & Responsibilities | Allowed Workspace Dependencies |
|---|---|---|---|
| `crates/wolio-diagnostics` | `T303`, `T306`, `T309`, `T310`, `T312`, `T316`, `T317` | Registers V3 diagnostic codes: `WOLIO-SECURITY-001` (`Secret<T>` violation or unvalidated provenance), `WOLIO-EFFECT-002` (unmocked external I/O or `Foreign::Call` in `test`/`property`), `WOLIO-RESOURCE-001` (aliased `resource` handle or missing `modifies` frame), `WOLIO-FOREIGN-001` (unauthorized foreign library), `WOLIO-FOREIGN-002` (storing foreign type in record/enum), `WOLIO-FUZZ-001` (property/contract fuzzing counterexample); enforces `Secret<T>` redaction (`"<REDACTED:Secret>"`) in all diagnostic payloads | none (leaf crate) |
| `crates/wolio-ast` | `T303`, `T309`, `T310`, `T312`, `T316`, `T317` | Adds `LanguageVersion::V3`, `Declaration::Property(PropertyDecl)`, `Declaration::ForeignType(ForeignTypeDecl)`, `Declaration::ForeignOp(ForeignOpDecl)`, `ParamMode::{Value, Resource}`, `ContractClause::Modifies(Vec<Spanned<String>>)`, `Expr::Old(Box<Expr>)`, `Expr::Transaction { handle: Spanned<String>, lets: Vec<LetStmt>, body: Box<Expr> }`, `Expr::WithMock { handlers: Vec<MockHandlerAst>, body: Box<Expr> }`, and fixed-width integer AST types | `wolio-diagnostics` |
| `crates/wolio-lexer` | `T303`, `T309`, `T316` | Tokenizes active keywords and 24 forward-reserved keywords across all versions (`foreign`, `with_mock`, `where`, `fn`, `secret`, `Secret`, `resource`, `transaction`, `modifies`, `old`, `property`, `std`, `Set`, etc.) | `wolio-diagnostics` |
| `crates/wolio-parser` | `T303`, `T309`, `T310`, `T312`, `T316`, `T317` | Accepts `LanguageVersion::{V1, V2, V3}`; under `V1`/`V2`, emits `WOLIO-FEATURE-001` on V3 constructs; under `V3`, parses `import std::<mod>;`, `foreign type ... from ...;`, `foreign operation ... from ... { ... }`, `resource <Type>` parameters, `modifies [...]`, `old(expr)`, `transaction (h) { ... }`, `with_mock { ... } in { ... }`, `property <name> for <op>(...) { where ...; ... }`, and fixed-width scalar types (`UInt8`..`UInt32`, `Int8`..`Int32`) | `wolio-lexer`, `wolio-ast`, `wolio-diagnostics` |
| `crates/wolio-semantic` | `T303`–`T313`, `T316`–`T318` | Unified compiler middle-end: resolves `std::*` & `foreign` symbols (`resolver`); type-checks `Secret<T>`, linear `resource`, linear `foreign type` (`WOLIO-FOREIGN-002`), fixed-width integers, `with_mock`, and `transaction` types (`types`); computes `sym_` IDs & hashes `[capabilities]`/`[foreign]` policies (`graph`); validates `modifies`, `old(...)`, `property`, and foreign operation contracts (`contracts`); enforces `[capabilities]`, `[foreign].allowed_libraries` (`WOLIO-FOREIGN-001`), `Foreign::Call` effect, `Secret` sinks, mandatory `with_mock` (`WOLIO-EFFECT-002`), and linear resource non-aliasing (`WOLIO-RESOURCE-001`) (`effects`); rejects vacuous properties and folds constants (`analyzer`) | `wolio-parser`, `wolio-ast`, `wolio-diagnostics` |
| `crates/wolio-verify` | `T301`, `T309`, `T311`, `T312`, `T318`, `T319` | Executes V2 `BUG-*` regressions; runs tests in hermetic sandbox rejecting unmocked live I/O & foreign calls (`WOLIO-EFFECT-002`); implements deterministic PRNG + boundary property fuzzer (`seed = 0x574F4C494F33`, 256 cases) and value-tree shrinker emitting `WOLIO-FUZZ-001` (`method: "property_fuzz"`) | `wolio-semantic`, `wolio-runtime`, `wolio-diagnostics` |
| `crates/wolio-runtime` | `T302`, `T304`–`T311`, `T314`, `T317`, `T318` | Implements Copy-on-Write (`Arc::make_mut` / `rc == 1`) `List<T>` and `Map<K, V>`; executes `std::*` modules, redacted `Secret<T>`, fixed-width integer checked arithmetic, scoped `with_mock` handler stacks, `Resource::acquire` / `Resource::mock`, foreign C glue ABI dispatch, `old(expr)` pre-state snapshots, and transactional resource adapters with automatic rollback on `Result::Err` or fault; exports V3 C runtime ABI | `wolio-ast`, `wolio-semantic`, `wolio-diagnostics` (NEVER LLVM) |
| `crates/wolio-ir` | `T302`, `T314`, `T317`, `T319` | Adds `IrType::{SecretPtr, ResourcePtr, ForeignPtr, UInt8, UInt16, UInt32, Int8, Int16, Int32}`, last-use ownership flags on `List`/`Map` mutations (`is_unique_hint`), `OldSnapshotCapture` entry instructions, foreign extern call regions, and verified `TransactionRegion` rollback blocks in `VerifiedWolioIr` | `wolio-semantic`, `wolio-diagnostics` (NEVER LLVM) |
| `crates/wolio-codegen` | `T302`, `T304`–`T308`, `T311`, `T314`, `T317`, `T319` | Lowers V3 `VerifiedWolioIr` instructions (`wolio_rt_list_push_cow`, `wolio_rt_map_insert_cow`, `wolio_rt_std_*`, `wolio_rt_tx_begin`/`commit`/`rollback`, foreign extern bindings with contract guards) to verified textual LLVM IR (`.ll`), maps fixed-width integers to native `i8`/`i16`/`i32`, and links via pinned `clang` | `wolio-ir`, `wolio-runtime`, `wolio-diagnostics` |
| `crates/wolio-package` | `T303` | Manages package dependencies, locked closures, and caching without semantic divergence | `wolio-semantic`, `wolio-diagnostics` |
| `crates/wolio-agent` | `T313` | Supports `protocol_version: 3` (`wolio.mutation.v3`) alongside v1 and v2; adds V3 AST schemas (`property`, `resource`, `modifies`, `old`, `transaction`, `with_mock`, `foreign type`, `foreign operation`, fixed-width integers); enforces `HUMAN_REQUIRED` on `[capabilities]`, `[foreign]`, `modifies`, and `Secret<T>` sink edits; runs `wolio verify --fuzz` before `apply`/`finalize`; redacts `Secret<T>` in `.wolio-log/` | `wolio-parser`, `wolio-semantic`, `wolio-verify`, `wolio-diagnostics` |
| `crates/wolio-format` | `T314` | Formats `property`, `resource` parameters, `foreign` types/ops, fixed-width integers, `modifies [...]`, `old(...)`, `transaction (h) { ... }`, and `with_mock { ... } in { ... }` canonically with 4-space LF indentation and `sym_` preservation check | `wolio-lexer`, `wolio-parser`, `wolio-ast`, `wolio-semantic`, `wolio-diagnostics` |
| `crates/wolio-lsp` | `T303`, `T314` | Incremental module-level semantic caching for `< 50 ms` diagnostics/hover on 100-module workspaces; hover, definition, references, and rename for `std::*`, `foreign` types/ops, fixed-width integers, `property`, `resource`, `old(...)`, and `with_mock` | `wolio-parser`, `wolio-semantic`, `wolio-verify`, `wolio-format`, `wolio-diagnostics` |
| `crates/wolio-cli` | `T303`, `T312`, `T315` | Dispatches `[language] version = "1" \| "2" \| "3"` across all commands and supports `wolio verify --fuzz [--fuzz-cases <N>] [--fuzz-seed <HEX>]` | All workspace service crates |

## 3. Core V3 representation and runtime ABI invariants

### 3.1 Copy-on-Write (CoW) reference-counting optimization for `List<T>` and `Map<K, V>` (`T302`)
- In V2 (`T207`/`T213`), `List::push`, `Map::insert`, and `Map::remove` cloned the underlying container on every call. In V3 (`T302`), collection values retain 100% pure immutable value semantics at the language level while mutating in place when the runtime reference count is uniquely owned (`rc == 1`):
  - **Interpreter (`wolio-runtime`):** `Value::List(Arc<Vec<Value>>)` and `Value::Map(Arc<BTreeMap<MapKey, Value>>)` use last-use binding consumption in `for` loop accumulators (`acc = List::push(acc, item)`) and local `let` rebindings so the moved `Arc` has `Arc::strong_count(&arc) == 1`, allowing `Arc::make_mut(&mut arc)` to mutate the `Vec` or `BTreeMap` in place in amortized `O(1)` / `O(log N)` time with zero full-collection clones. When an alias exists (`Arc::strong_count(&arc) > 1`), `Arc::make_mut` clones the underlying container before mutating, preserving all prior aliases unchanged.
  - **Native C Runtime ABI (`wolio-runtime` / `wolio-codegen`):** Heap collection headers carry an atomic 64-bit reference count (`rc: AtomicU64`). `wolio_rt_list_push_cow(list_ptr, elem)`, `wolio_rt_map_insert_cow(map_ptr, key, val)`, and `wolio_rt_map_remove_cow(map_ptr, key)` inspect `rc`: if `rc == 1`, they mutate the buffer/B-tree in place and return `list_ptr`/`map_ptr`; if `rc > 1`, they allocate a shallow copy (incrementing element refcounts), decrement the old container's `rc`, and mutate the new copy.
  - **Zero-leak invariant:** Every scope exit, early `Result::Err` return, loop iteration step, and `transaction` rollback decrements collection reference counts deterministically, verified by an atomic allocation/deallocation counter in `wolio-runtime` asserting `live_allocations == 0` on program/test exit.

### 3.2 Industrial standard library (`std::*`) (`T304`, `T305`, `T307`, `T308`)
- Every `std::<module>` namespace requires an explicit `import std::<module>;` at the top of the `.wol` file and explicit `dependencies [std::<module>::<op>, ...]` (and `effects [...]` for effectful operations) in every calling `operation`.
- **Pure modules (`effects []`):**
  - `std::string`: `len(String) -> Int` (UTF-8 byte length), `slice(String, Int, Int) -> Option<String>` (returns `Option::None` if out of bounds or not on UTF-8 char boundaries), `contains(String, String) -> Bool`, `starts_with(String, String) -> Bool`, `ends_with(String, String) -> Bool`, `trim(String) -> String`, `split(String, String) -> List<String>`, `parse_int(String) -> Result<Int, StringError>`, `parse_decimal(String) -> Result<Decimal, StringError>`.
  - `std::math`: `abs(Int) -> Int` (faults with `WOLIO-RUNTIME-001` on `i64::MIN`), `min(Int, Int) -> Int`, `max(Int, Int) -> Int`, `clamp(Int, Int, Int) -> Int` (faults with `WOLIO-CONTRACT-002` if `lo > hi`), `pow_checked(Int, UInt) -> Result<Int, MathError>`.
  - `std::collections`: `sort_int(List<Int>) -> List<Int>`, `dedup_int(List<Int>) -> List<Int>`, `reverse<T>(List<T>) -> List<T>`, `zip<A, B>(List<A>, List<B>) -> List<Pair<A, B>>`, `take<T>(List<T>, Int) -> List<T>`, `drop<T>(List<T>, Int) -> List<T>`.
  - `std::json`: `Value` enum (`Null`, `Bool(Bool)`, `Int(Int)`, `Decimal(Decimal)`, `String(String)`, `Array(List<Value>)`, `Object(Map<String, Value>)` — zero IEEE-754 floats), `parse(String) -> Result<Value, JsonError>`, `stringify(Value) -> String` (deterministic compact RFC 8785 / canonical lexicographical key ordering).
- **Cryptographic module (`std::crypto`):**
  - `sha256(Bytes) -> Bytes` (pure, `effects []`, returns 32-byte digest).
  - `constant_time_eq(Bytes, Bytes) -> Bool` (pure, `effects []`, constant-time byte comparison).
  - `hmac_sha256(Secret<Bytes>, Bytes) -> Bytes` (requires `effects [Secret::Use]`, returns 32-byte MAC without exposing the secret key).
- **Capability-gated I/O modules:**
  - `std::time`: `now() -> Timestamp` (`effects [Clock::Read]`).
  - `std::fs`: `read_text(Path) -> Result<String, FsError>` (`effects [Fs::Read]`), `write_text(Path, String) -> Result<Unit, FsError>` (`effects [Fs::Write]`).
  - `std::net`: `http_send(HttpRequest) -> Result<HttpResponse, NetError>` (`effects [Net::Http]`).
  - `std::env`: `read_secret(String) -> Result<Secret<String>, EnvError>` (`effects [Secret::Use]`).

### 3.3 Opaque `Secret<T>` and input validation provenance (`WOLIO-SECURITY-001`, `T306`)
- `Secret<T>` is restricted to `T = String` or `T = Bytes` (other type arguments fail with `WOLIO-TYPE-001`).
- **Compile-time sink & construction rules (`WOLIO-SECURITY-001`):**
  1. Constructing `Secret` from a string/bytes literal or pure expression in a production module is rejected with `WOLIO-SECURITY-001` (only `std::env::read_secret`, capability-gated secret loaders, or `Secret::for_test(...)` inside `test`/`property` blocks may construct a `Secret<T>`).
  2. Accessing `.value`, applying `==` or `!=`, or passing `Secret<T>` (or any record/enum/collection transitively containing `Secret<T>`) to `console::print`, `format(...)`, `std::json::stringify`, `std::fs::write_text`, or `std::net::http_send` (except in dedicated `Secret::Use` auth headers/crypto sinks) fails with `WOLIO-SECURITY-001`.
  3. Runtime diagnostics, `.wolio-log/` mutation/replay journals, and contract failure dumps always format any `Secret<T>` value as `"<REDACTED:Secret>"` and zeroize backing memory on drop.
- **External input validation provenance (`WOLIO-SECURITY-001`):**
  - Values originating from external untrusted sources (`std::json::parse`, `std::fs::read_text`, `std::net::http_send`) carry an `UntrustedOrigin` provenance tag in `wolio-semantic::effects`.
  - Constructing a domain `type` (nominal wrapper) directly from an `UntrustedOrigin` value inside an operation that does not guarantee a validating `predicate` over `result` in its `ensures` clause fails static checking with `WOLIO-SECURITY-001`. External data must pass through a smart-constructor operation (`fn/op ParseX(raw: String) -> Result<DomainX, Err>` with `ensures match (result) { Result::Ok(v) => ValidDomainX(v), ... }`).

### 3.4 Capability-gated I/O and hermetic `with_mock` test sandbox (`WOLIO-EFFECT-002`, `T308`, `T309`)
- **`wolio.toml` `[capabilities]`:**
  ```toml
  [capabilities]
  allowed_effects = ["Console::Write", "Clock::Read", "Fs::Read", "Net::Http", "Db::Tx", "Secret::Use", "Foreign::Call"]
  ```
  If any operation in a package declares an effect in `effects [...]` that is not listed in `[capabilities].allowed_effects` (default `["Console::Write"]` when `[capabilities]` is omitted), `wolio check` fails with `WOLIO-EFFECT-001`. Modifying `[capabilities]` requires `HUMAN_REQUIRED` approval.
- **Hermetic test and property sandbox (`WOLIO-EFFECT-002`):**
  - `wolio test` and `wolio verify` forbid live external I/O (`Clock::Read`, `Fs::Read`, `Fs::Write`, `Net::Http`, `Db::Query`, `Db::Tx`, `Secret::Use`, `Foreign::Call`).
  - Any `test` or `property` block calling an operation whose transitive `effects` include a non-console effect must wrap the call in:
    ```wol
    with_mock {
        std::time::now => fn() -> Timestamp { Timestamp(1700000000) },
        std::net::http_send => fn(req: HttpRequest) -> Result<HttpResponse, NetError> { ... }
    } in {
        MyOperation()
    }
    ```
  - Static check in `wolio-semantic::effects` and runtime sandbox guard in `wolio-runtime` reject any unmocked external effect inside `test` or `property` with `WOLIO-EFFECT-002`. Mock handler bodies must be pure (`effects []`).

### 3.5 Linear `resource` handles, acquisition, `modifies [...]`, `old(expr)`, and `transaction` rollback (`WOLIO-RESOURCE-001`, `T310`, `T311`)
- **Scope-bound lexical acquisition (deadlock resolution):**
  - `resource <Type>` instances cannot be created from raw values or literals.
  - Production acquisition: `Resource::acquire::<T>(config)` can be called only in package `main` or authorized capability adapter operations.
  - Test acquisition: `Resource::mock::<T>()` can be called only within an enclosing `with_mock` block.
- **Linear non-aliasing rules (`WOLIO-RESOURCE-001`):**
  - Parameters declared as `name: resource HandleType` cannot be aliased: passing the same resource variable in two argument positions of a single call (`Transfer(db, db, ...)`) or creating aliased bindings (`let db2 = db;`) fails statically with `WOLIO-RESOURCE-001`.
  - Capturing a resource handle inside a `record`, `enum`, `Option`, `Result`, `List`, or `Map`, or returning a resource without explicit linear transfer capability fails statically with `WOLIO-RESOURCE-001`.
- **Frame condition `modifies [handle, ...]` (`WOLIO-RESOURCE-001`):**
  - Any operation that mutates a `resource` handle directly or passes it to a callee that mutates it must declare `modifies [handle];` in its contract block and include the corresponding resource effect (e.g., `Db::Tx`); omitting `modifies [handle];` fails with `WOLIO-RESOURCE-001`.
- **Pre-state `old(expr)` snapshots (`T310`):**
  - Inside `ensures`, `old(expr)` may wrap any pure expression/predicate that reads a `resource` handle declared in `modifies [...]`.
  - Before the operation body executes, `wolio-runtime` (and native codegen) evaluates each distinct `old(expr)` in source order and binds the immutable snapshot value for postcondition evaluation after body completion.
- **Atomic `transaction (handle) { ... }` blocks (`T311`):**
  - `transaction (handle) { ... }` requires `handle` to be an in-scope `resource` parameter listed in `modifies [handle]` and its body to have type `Result<T, E>`.
  - On entry, the runtime captures a full transactional state checkpoint of `handle` (`tx_begin`).
  - If the body evaluates to `Result::Ok(v)`, the runtime commits the checkpoint (`tx_commit`) and returns `Result::Ok(v)`.
  - If the body evaluates to `Result::Err(e)` or triggers a fault, the runtime atomically restores `handle` to its entry checkpoint (`tx_rollback`) **before** evaluating the operation's `ensures` clause, guaranteeing that error-branch postconditions (`Result::Err(_) => Balance(db, id) == old(Balance(db, id))`) hold even when intermediate steps mutated `db` before failing.

### 3.6 Contract-driven property fuzzer and deterministic shrinker (`WOLIO-FUZZ-001`, `T312`)
- **Generator (`crates/wolio-verify`):**
  - Seeded deterministically with `DEFAULT_FUZZ_SEED = 0x574F4C494F33` (`"WOLIO3"` in ASCII hex) XORed with the target symbol's 64-bit `sym_` prefix so fuzzing is 100% reproducible across machines and runs.
  - For every `property <name> for <op>(<params>) { where <pred>; <body> }` and every operation with `requires`/`ensures` when `wolio verify --fuzz` runs:
    1. Synthesizes structured boundary values first (`0`, `1`, `-1`, `i64::MIN`, `i64::MAX`, boundary constants extracted from `requires`/`where` comparisons `x >= C` -> `C, C + 1`, empty/singleton `String`/`List`/`Map`, `Decimal` scale boundaries).
    2. Synthesizes deterministic SplitMix64 PRNG values up to `N = 256` valid inputs satisfying `requires` and `where` (filtering out inputs where `requires`/`where` is `false`, and failing with `WOLIO-CONTRACT-001` if `< 16` valid inputs are found within `4096` attempts due to an unsatisfiable precondition).
- **Deterministic Value-Tree Shrinker:**
  - When a generated input `x` causes `ensures` to return `false`, the `property` body to return `false`, or an unhandled runtime fault (`WOLIO-RUNTIME-001` / `WOLIO-CONTRACT-004`), the shrinker deterministically searches smaller candidates (binary-searching integers/decimals toward `0` or the lower precondition bound, truncating strings/lists/maps from the tail, simplifying enum variants) while preserving `requires`/`where == true`.
  - Emits `WOLIO-FUZZ-001` with `violation: { "seed": "0x574F4C494F33", "counterexample": { ... }, "shrunk_steps": k, "failed_clause": ... }` and records a `property_fuzz` evidence item (`EVIDENCE_MODEL.md`).

### 3.7 Capability-Gated Foreign Function Interface (FFI) & C Glue Generation (`T316`–`T319`)
- **Manifest gating (`WOLIO-FOREIGN-001`):**
  - `wolio.toml` defines `[foreign] allowed_libraries = ["libhardware", "libsqlite3"]`.
  - Any `foreign type ... from "<lib>";` or `foreign operation ... from "<lib>"` where `<lib>` is not in `allowed_libraries` triggers compile-time error `WOLIO-FOREIGN-001`.
  - Modifying `[foreign].allowed_libraries` requires `HUMAN_REQUIRED` approval.
- **Linear opaque `foreign type` (`WOLIO-FOREIGN-002`):**
  - Type-checked as an opaque linear handle.
  - Storing a foreign type in fields of ordinary `record` types, `enum` payloads, `List`, `Map`, `Option`, or `Result` triggers `WOLIO-FOREIGN-002`.
  - Auto-drop glue: On leaving its lexical scope, the compiler automatically emits a call to `<lib>_<type>_drop(handle)` in `wolio-runtime` and LLVM IR, preventing memory/handle leaks.
- **Contract-guarded `foreign operation` and error code marshalling (`T318`):**
  - Every foreign operation declares `requires`, `ensures`, `effects [Foreign::Call]`, and `errors [ForeignError::...]`.
  - Compiler generates C glue wrapper functions that:
    1. Pre-check `requires` before crossing the C boundary (faulting with `WOLIO-CONTRACT-002` if violated).
    2. Convert native Wolio values (fixed-width integers, opaque pointers, strings) to C ABI types.
    3. Invoke the C library symbol.
    4. Marshall return status / C `errno` into structured `Result<T, ForeignError>`.
    5. Post-check `ensures` postconditions upon return (faulting with `WOLIO-CONTRACT-003` if violated).
  - AI agents never write raw pointers or `unsafe` blocks; the boundary is completely compiler-generated and contract-verified.
- **Native LLVM IR lowering & linking (`T319`):**
  - `crates/wolio-codegen` emits textual LLVM IR (`.ll`) declaring `declare` extern C functions for foreign symbols and generates intermediate wrappers enforcing contract assertions.
  - Linked via pinned `clang` using `-l<lib>` or direct static object linking against libraries listed in `allowed_libraries`.
  - Sandbox test execution: Invoking any operation with effect `Foreign::Call` in `wolio test` or `wolio verify` without a corresponding `with_mock` handler triggers `WOLIO-EFFECT-002`.

### 3.8 Fixed-Width Integer Scalar Types (`UInt8`..`UInt32`, `Int8`..`Int32`) (`T317`, `T319`)
- **Native bit width & LLVM mapping:**
  - `UInt8` / `Int8` -> LLVM `i8`
  - `UInt16` / `Int16` -> LLVM `i16`
  - `UInt32` / `Int32` -> LLVM `i32`
- **Strict checked arithmetic:**
  - Arithmetic operators `+`, `-`, `*`, `/`, `%` perform checked arithmetic within the exact range of the type.
  - Any overflow, underflow, or divide by zero triggers an immediate runtime fault (`WOLIO-RUNTIME-001` or `WOLIO-CONTRACT-004`).
  - Zero silent truncation, wraparound, or undefined behavior.
- **Explicit conversions:**
  - Narrowing constructors: `UInt8::from_uint(x)` (faults if `x > 255`), `UInt16::from_uint(x)` (faults if `x > 65535`), `UInt32::from_uint(x)` (faults if `x > 4294967295`), `Int8::from_int(x)` (faults if `x < -128 || x > 127`), `Int16::from_int(x)` (faults if `x < -32768 || x > 32767`), `Int32::from_int(x)` (faults if `x < -2147483648 || x > 2147483647`).
  - Widening operations: `.to_uint()` / `.to_int()`.
  - No implicit widening or narrowing coercions.


