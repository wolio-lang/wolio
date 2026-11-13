`.blueprint/04-tasks/V2-TASKS.md`
# V2 Task Contracts (`M08`–`M11`: `T201`–`T216`)

Normative overview and quick-reference index for [BASELINE-V2.md](../BASELINE-V2.md),
[CORE_V2.md](../02-specifications/language/CORE_V2.md), and
[IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md).
Every V2 task (`T201`–`T216`) has its own complete, standalone self-contained
contract file linked below:

- **M08 (`T201`–`T203`):** [T201-v1-defect-triage-regressions.md](T201-v1-defect-triage-regressions.md), [T202-v1-defect-remediation.md](T202-v1-defect-remediation.md), [T203-language-version-isolation-gate.md](T203-language-version-isolation-gate.md)
- **M09 (`T204`–`T208`):** [T204-v2-loop-predicate-parser.md](T204-v2-loop-predicate-parser.md), [T205-uint-bytes-decimal-scalars.md](T205-uint-bytes-decimal-scalars.md), [T206-bounded-for-loop-guards.md](T206-bounded-for-loop-guards.md), [T207-immutable-list-canonical-map.md](T207-immutable-list-canonical-map.md), [T208-deterministic-typechecked-format.md](T208-deterministic-typechecked-format.md)
- **M10 (`T209`–`T212`):** [T209-reusable-pure-predicates.md](T209-reusable-pure-predicates.md), [T210-architecture-module-policy.md](T210-architecture-module-policy.md), [T211-agent-protocol-v2-modules.md](T211-agent-protocol-v2-modules.md), [T212-v2-anti-workaround-analyzer.md](T212-v2-anti-workaround-analyzer.md)
- **M11 (`T213`–`T216`):** [T213-v2-ir-llvm-native-lowering.md](T213-v2-ir-llvm-native-lowering.md), [T214-v2-formatter-lsp-parity.md](T214-v2-formatter-lsp-parity.md), [T215-v2-algorithmic-benchmark.md](T215-v2-algorithmic-benchmark.md), [T216-v2-release-distribution.md](T216-v2-release-distribution.md)

These tasks become eligible only after V1 (`M01`–`M07`) completion and owner
acceptance of the V2 execution start.

---

## M08 — V1 Bug-Fix, Regression Hardening & Migration Gate (`T201`–`T203`)

### T201: Triage V1 benchmark/implementation defects and build `BUG-*` regression suite
- **Depends on:** `T063`, `T074` (and owner V2 gate approval) | **Blocks:** `T202`
- **Bounded areas:** `tests/conformance/regressions/`, `.execution/reports/BUG-*.md`
- **Acceptance families:** `V2-REGRESSION`
- **Self-contained contract:**
  - Audit all failure records, escaped defects, and compiler edge cases from V1 (`M01`–`M07`, especially `T034` and `T062`).
  - For every identified defect, create a minimal reproducible conformance fixture under `tests/conformance/regressions/BUG-nnnn/` asserting the exact expected diagnostic or runtime output.
  - Verify that the test runner executes all `BUG-nnnn` fixtures deterministically.

### T202: Fix all V1 compiler, protocol, runtime, and native codegen defects
- **Depends on:** `T201` | **Blocks:** `T203`
- **Bounded areas:** `crates/wolio-*`
- **Acceptance families:** `V2-REGRESSION`, all V1 families (`FRONT`..`TOOL`)
- **Self-contained contract:**
  - Fix every root cause reproduced in `T201` without weakening any V1 contract, analyzer rule, or acceptance case.
  - Ensure both the interpreter (`wolio-runtime`) and LLVM backend (`wolio-codegen`) pass 100% of `tests/conformance/regressions/BUG-*`.

### T203: Enforce `[language] version = "1" | "2"` isolation gate
- **Depends on:** `T202` | **Blocks:** `T204`
- **Bounded areas:** `crates/wolio-cli`, `crates/wolio-parser`, `crates/wolio-semantic`
- **Acceptance families:** `V2-REGRESSION`
- **Self-contained contract:**
  - Accept `version = "1"` and `version = "2"` in `wolio.toml` `[language]`.
  - Guarantee that projects with `version = "1"` reject any V2 construct with `WOLIO-FEATURE-001` and produce bit-identical `sym_` hashes, snapshot hashes, and diagnostics to V1.

---

## M09 — Total Bounded Loops, Immutable Collections & Exact Decimal (`T204`–`T208`)

### T204: Parse `CORE_V2` bounded `for` loops, range expressions, and `predicate` declarations
- **Depends on:** `T203` | **Blocks:** `T205`, `T206`, `T209`
- **Bounded areas:** `crates/wolio-lexer`, `crates/wolio-ast`, `crates/wolio-parser`
- **Acceptance families:** `V2-LOOP` (syntax)
- **Self-contained contract:**
  - Promote reserved V2 tokens (`in`, `with`, `bound`, `invariant`, `predicate`, `List`, `Map`, `Decimal`, `UInt`, `Bytes`, `format`, `..`) to active grammar terminals when `[language] version = "2"`.
  - Parse `for (item in source with acc: Type = init) { bound expr; invariant expr; lets...; body_expr }` and `predicate Name(fields) -> Bool { expr }` into spanned AST nodes.
  - Reject loops missing either `bound` or `invariant` with `WOLIO-SYNTAX-001`.

### T205: Implement `UInt`, `Bytes`, and exact 128-bit fixed-scale `Decimal`
- **Depends on:** `T204` | **Blocks:** `T206`, `T207`, `T208`
- **Bounded areas:** `crates/wolio-types`, `crates/wolio-runtime`
- **Acceptance families:** `V2-DATA` (scalars)
- **Self-contained contract:**
  - Implement 64-bit unsigned `UInt` (checked `+ - * / %`, underflow/overflow faults with `WOLIO-RUNTIME-001`).
  - Implement immutable `Bytes` value representation and structural equality.
  - Implement exact signed 128-bit fixed-point `Decimal` (scale `10^6`): constructors `Decimal::of(Int)` and `Decimal::from_scaled(Int, Int)`, checked arithmetic, exact comparison, and runtime fault (`WOLIO-RUNTIME-001`) on overflow or division by zero. No binary float (`f32`/`f64`) is used anywhere.

### T206: Type-check and execute total bounded `for` loops with `bound` and `invariant` guards
- **Depends on:** `T205` | **Blocks:** `T207`
- **Bounded areas:** `crates/wolio-types`, `crates/wolio-contracts`, `crates/wolio-runtime`
- **Acceptance families:** `V2-LOOP`
- **Self-contained contract:**
  - Enforce that `bound` and `invariant` expressions have type `Bool` and contain no operation calls (`WOLIO-CONTRACT-001`).
  - Enforce that `item` and `acc` do not shadow visible bindings (`WOLIO-RESOLVE-001`) and the loop body type matches the declared accumulator type (`WOLIO-TYPE-001`).
  - At runtime: evaluate `bound` once before entry (aborting with `WOLIO-CONTRACT-005` if `false`); evaluate `invariant` on entry (`acc = init`), after each iteration (`acc = next_acc`), and on exit (aborting with `WOLIO-CONTRACT-006` if `false`).

### T207: Implement immutable `List<T>` and canonical-order `Map<K, V>`
- **Depends on:** `T206` | **Blocks:** `T208`
- **Bounded areas:** `crates/wolio-types`, `crates/wolio-runtime`
- **Acceptance families:** `V2-DATA` (collections)
- **Self-contained contract:**
  - Type-check `List<T>` and `Map<K, V>` (`K` restricted to `Int`, `UInt`, `String`, `Bool`, or nominal types over them; other key types fail with `WOLIO-TYPE-001`).
  - Implement pure value-semantic operations: `List::empty`, `List::push`, `List::len`, `List::get` (returning `Option<T>`), and `Map::empty`, `Map::insert`, `Map::get`, `Map::remove`, `Map::len`, `Map::keys` (sorted in canonical key order).
  - Support `for (item in list_expr with acc: T = init)` iteration over `List<T>`.

### T208: Implement type-checked deterministic `format(template, ...)`
- **Depends on:** `T207` | **Blocks:** `T209`
- **Bounded areas:** `crates/wolio-types`, `crates/wolio-runtime`
- **Acceptance families:** `V2-DATA` (`format`)
- **Self-contained contract:**
  - Validate at compile time that the first argument to `format(...)` is a string literal whose `{}` placeholder count equals the remaining argument count (`WOLIO-TYPE-001`).
  - Accept only `Int`, `UInt`, `Decimal`, `Bool`, or `String` arguments; reject unextracted nominal types (requiring `.value`), records, enums, or collections (`WOLIO-TYPE-001`).

---

## M10 — Pure Predicates, Architecture Policy & Agent Protocol v2 (`T209`–`T212`)

### T209: Implement reusable pure `predicate` declarations and contract calls
- **Depends on:** `T208` | **Blocks:** `T210`
- **Bounded areas:** `crates/wolio-resolver`, `crates/wolio-semantic`, `crates/wolio-contracts`, `crates/wolio-runtime`
- **Acceptance families:** `V2-PRED`
- **Self-contained contract:**
  - Assign deterministic `sym_` IDs with kind `"predicate"` (`wolio.symbol.v1`).
  - Enforce that `predicate` bodies have type `Bool`, call only other `predicate`s and pure builtin constructors/collection accessors, and form a strict DAG (rejecting predicate call cycles with `WOLIO-RESOLVE-001` and operation calls with `WOLIO-CONTRACT-001`).
  - Allow `requires`, `ensures`, `bound`, and `invariant` clauses to call visible `predicate`s.

### T210: Enforce `[architecture]` module dependency allow/deny rules (`WOLIO-ARCH-001`)
- **Depends on:** `T209` | **Blocks:** `T211`
- **Bounded areas:** `crates/wolio-effects`, `crates/wolio-semantic`
- **Acceptance families:** `V2-ARCH`
- **Self-contained contract:**
  - Parse `[architecture]` `allow_imports` and `deny_imports` rules in `wolio.toml` and include them in the project policy snapshot hash.
  - Check both direct `import` declarations and transitive module dependency paths; any denied edge or edge omitted from an active allowlist fails with `WOLIO-ARCH-001`.

### T211: Upgrade Agent Protocol to v2 (`create_module`, `delete_module`, `add_import`, `remove_import`)
- **Depends on:** `T210` | **Blocks:** `T212`
- **Bounded areas:** `crates/wolio-agent`
- **Acceptance families:** `V2-AGENT`
- **Self-contained contract:**
  - Support `protocol_version: 2` alongside `protocol_version: 1`.
  - Add typed AST payloads for `for` loops, `predicate` declarations, `List`/`Map`/`Decimal`/`UInt`/`Bytes`, and `format`.
  - Add `create_module`, `delete_module`, `add_import`, and `remove_import` plan operations, computing `POST_REVIEW` or `HUMAN_REQUIRED` tiers according to public visibility and `[architecture]` policy.

### T212: Extend anti-workaround analyzer for V2 loops and predicates
- **Depends on:** `T211` | **Blocks:** `T213`
- **Bounded areas:** `crates/wolio-analyzer`
- **Acceptance families:** `V2-AGENT`, `ANALYZER`
- **Self-contained contract:**
  - Reject tautological loop invariants (`invariant true;`) in production implementations with `WOLIO-ANALYZER-001` (treating `invariant true;` as a verification bypass).
  - Extend `WOLIO-ANALYZER-004` constant-folding to `Decimal`, `UInt`, and `format(...)` expressions.

---

## M11 — V2 Native Parity, Tooling & Algorithmic Benchmark (`T213`–`T216`)

### T213: Lower V2 bounded loops, collections, predicates, and `Decimal` to `VerifiedWolioIr` and LLVM
- **Depends on:** `T212` | **Blocks:** `T214`
- **Bounded areas:** `crates/wolio-ir`, `crates/wolio-codegen`, `crates/wolio-runtime`
- **Acceptance families:** `V2-NATIVE`
- **Self-contained contract:**
  - Extend `VerifiedWolioIr` with verified bounded loop headers, `bound`/`invariant` guard blocks (`WOLIO-CONTRACT-005`/`006`), 128-bit `Decimal` ops, and reference-counted `List`/`Map` runtime ABI calls.
  - Prove 100% differential parity between the interpreter and release-optimized native builds across all `V2-*` conformance fixtures.

### T214: Update `wolio fmt` and `wolio lsp` for all `CORE_V2` constructs
- **Depends on:** `T213` | **Blocks:** `T215`
- **Bounded areas:** `crates/wolio-format`, `crates/wolio-lsp`
- **Acceptance families:** `V2-NATIVE`, `TOOL`
- **Self-contained contract:**
  - Format `predicate` and `for..in..with..bound..invariant` canonically with idempotence and `sym_` preservation.
  - Support hover, go-to-definition, references, and rename for `predicate` declarations and V2 types in `wolio lsp`.

### T215: Execute 24-task V2 algorithmic, collection, and architecture benchmark (`W04`, `W07`, `W08`)
- **Depends on:** `T214` | **Blocks:** `T216`
- **Bounded areas:** `benchmarks/`
- **Acceptance families:** `V2-BENCH`
- **Self-contained contract:**
  - Freeze and run 24 algorithmic, collection-processing, exact-financial (`Decimal`), and multi-module architecture tasks × 10 paired seeds across Wolio V2, Go, Rust, and TypeScript.
  - Compute 10,000-resample paired bootstrap 95% confidence intervals and publish raw records and negative findings.

### T216: Package and verify Wolio V2 Linux x86_64 release distribution
- **Depends on:** `T215` | **Blocks:** `T301` (V3 entry)
- **Bounded areas:** release packaging, CI
- **Acceptance families:** `V2-NATIVE`, `TOOL`
- **Self-contained contract:**
  - Package the V2 Linux x86_64 `.tar.gz` release archive with SHA-256 checksums, verify clean-machine installation, and confirm both `version = "1"` and `version = "2"` sample workflows pass.

