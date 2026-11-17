`.blueprint/03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md`
# Wolio implementation specification V2 (`M08`–`M11`)

Status: revision 1.5 freeze candidate. This document is the authoritative
crate-level implementation specification for Wolio V2 (`[language] version = "2"`,
milestones `M08`–`M11`, tasks `T201`–`T216`). It extends
[IMPLEMENTATION_SPECIFICATION_V1.md](IMPLEMENTATION_SPECIFICATION_V1.md) without
introducing new Cargo crates or breaking V1 (`[language] version = "1"`) isolation,
and is further extended in V3 (`M12`–`M15`, `T301`–`T319`) by
[IMPLEMENTATION_SPECIFICATION_V3.md](IMPLEMENTATION_SPECIFICATION_V3.md).

## 1. V2 compiler and tooling pipeline extensions

```text
source (.wol) + wolio.toml ([language] version = "1" | "2", [architecture])
  -> wolio-lexer (promotes V2 reserved tokens in/with/bound/invariant/predicate/List/Map/Decimal/UInt/Bytes/format, keeps 23 V2/V3 keywords reserved, and adds `..` range token)
  -> wolio-parser / wolio-ast (parses PredicateDecl, ForExpr with mandatory bound/invariant, IterSource::Range, turbofish generic calls path::<TArgs>(...); gates V2 AST behind language version "2")
  -> wolio-semantic (unified middle-end extending 6 submodules):
       ├── resolver: resolves predicate symbols, checks predicate call DAG for cycles, rejects item/acc loop shadowing
       ├── types: type-checks UInt, Bytes, 128-bit fixed-scale Decimal, RoundingMode, checked bitwise ops (bit_and, bit_or, bit_xor, bit_shl, bit_shr, bit_not), List<T>, Map<K,V>, format(...) template arity/types, and ForExpr accumulator types
       ├── graph: computes sym_ IDs with kind "predicate", hashes [architecture] policy into snapshot hash, adds calls_predicate edges
       ├── contracts: validates predicate purity, allows predicate calls inside requires/ensures/bound/invariant, rejects operation calls with WOLIO-CONTRACT-001
       ├── effects: enforces [architecture] allow_imports / deny_imports across direct and transitive module imports with WOLIO-ARCH-001
       └── analyzer: rejects tautological `invariant true;` with WOLIO-ANALYZER-001; folds Decimal/UInt/format constants for WOLIO-ANALYZER-004; detects UInt/Decimal test-literal mirroring (WOLIO-ANALYZER-005)
  -> wolio-verify (executes BUG-* regression suite, runs V2 loop/collection/predicate/architecture obligations, and scalar fuzzing)
  -> wolio-runtime (evaluates UInt/Bytes/Decimal, explicit RoundingMode, bitwise ops, value-semantic List/Map, format, predicate calls, and bounded for loops with WOLIO-CONTRACT-005/006 guards)
  -> wolio-ir (lowers bounded loops to verified loop header/latch/guard blocks, 128-bit Decimal ops, bitwise ops, and collection ABI calls in VerifiedWolioIr)
  -> wolio-codegen (emits verified textual LLVM IR .ll for V2 VerifiedWolioIr instructions and compiles via pinned clang toolchain)
```

Tooling extensions in V2:
- `wolio-agent`: adds `protocol_version: 2` (`wolio.mutation.v2`) supporting `create_module`, `delete_module`, `add_import`, `remove_import`, canonical Wolio source snippet strings (`code: string`) parsed server-side into verified AST diffs, and V2 AST payloads (`for`, `predicate`, `List`, `Map`, `Decimal`, `UInt`, `Bytes`, `format`).
- `wolio-format`: formats `predicate` declarations, `for (item in src with acc: T = init) { bound ...; invariant ...; ... }` blocks, and turbofish constructor calls canonically while verifying `sym_` ID preservation.
- `wolio-lsp`: adds semantic tokens, hover, go-to-definition, find-references, and workspace rename for `predicate` declarations, loop bindings, turbofish calls, and V2 builtin types.

## 2. Crate-by-crate V2 implementation responsibilities (`T201`–`T216`)

No new Cargo crates are created in V2; all V2 functionality extends the existing
14-crate DAG established in V1 (`IMPLEMENTATION_SPECIFICATION_V1.md`):

| Crate | V2 Tasks | V2 Data Structures & Responsibilities | Allowed Workspace Dependencies |
|---|---|---|---|
| `crates/wolio-diagnostics` | `T203`, `T206`, `T210` | Registers V2 diagnostic codes: `WOLIO-CONTRACT-005` (loop bound violation), `WOLIO-CONTRACT-006` (loop invariant violation), `WOLIO-ARCH-001` (module architecture policy violation) | none (leaf crate) |
| `crates/wolio-ast` | `T204` | Adds `Declaration::Predicate(PredicateDecl)`, `Expr::For(ForExpr)`, `IterSource::{Collection(Expr), Range { start: Expr, end: Expr }}`, `Expr::Format { template: Spanned<String>, args: Vec<Expr> }`, and turbofish type args in `Expr::Call` | `wolio-diagnostics` |
| `crates/wolio-lexer` | `T204` | Adds `TokenKind::DotDot` (`..`); keeps all 24 V2/V3 keywords tokenized as distinct keyword variants across versions | `wolio-diagnostics` |
| `crates/wolio-parser` | `T203`, `T204` | Accepts `LanguageVersion::{V1, V2}`; under `V1`, emits `WOLIO-FEATURE-001` when V2 declarations/expressions/types are parsed; under `V2`, parses `predicate`, `for..in..with..bound..invariant`, and turbofish generic calls (`path::<TArgs>(...)`) | `wolio-lexer`, `wolio-ast`, `wolio-diagnostics` |
| `crates/wolio-semantic` | `T203`, `T205`–`T210`, `T212` | Extends unified middle-end: resolves `predicate` symbols & checks acyclic graph (`resolver`); type-checks `UInt`, `Bytes`, `Decimal`, `RoundingMode`, checked bitwise ops, `List`, `Map`, `Set`, `format` (`types`); computes `sym_` IDs & hashes `[architecture]` policy (`graph`); checks `predicate` purity & contract calls (`contracts`); enforces `[architecture]` rules (`effects`); rejects tautological loop invariants & folds constants (`analyzer`) | `wolio-parser`, `wolio-ast`, `wolio-diagnostics` |
| `crates/wolio-verify` | `T201`–`T203`, `T206`, `T210`, `T212` | Runs `tests/conformance/regressions/BUG-*` suite; records `runtime_guard` obligations for loop `bound` and `invariant` clauses and `static` obligations for `[architecture]` policy | `wolio-semantic`, `wolio-runtime`, `wolio-diagnostics` |
| `crates/wolio-runtime` | `T205`–`T209`, `T213` | Adds `Value::{UInt(u64), Bytes(Arc<[u8]>), Decimal(i128), List(Arc<Vec<Value>>), Map(Arc<BTreeMap<MapKey, Value>>), Set(Arc<BTreeSet<MapKey>>)}`; implements checked bitwise ops, `Decimal::mul_round`/`div_round`; executes bounded `for` loops checking `bound` (`WOLIO-CONTRACT-005`) and entry/step/exit `invariant` (`WOLIO-CONTRACT-006`); exports C ABI functions for native V2 codegen | `wolio-ast`, `wolio-semantic`, `wolio-diagnostics` (NEVER LLVM) |
| `crates/wolio-ir` | `T213` | Adds `IrType::{UInt64, Decimal128, BytesPtr, ListPtr, MapPtr, SetPtr}`, `IrInst::{DecimalArith, DecimalRound, Bitwise, BoundGuard, InvariantGuard, CollectionCall}`, and verified bounded loop CFG regions (`LoopHeader`, `LoopBody`, `LoopStepGuard`, `LoopExit`) to `VerifiedWolioIr` | `wolio-semantic`, `wolio-diagnostics` (NEVER LLVM) |
| `crates/wolio-codegen` | `T213` | Lowers `VerifiedWolioIr` V2 instructions (`i128` checked decimal arithmetic, decimal rounding, bitwise operations, bounded loop basic blocks, and `wolio_rt_*` C ABI calls for `Bytes`/`List`/`Map`/`Set`/`format`) to verified textual LLVM IR (`.ll`) and compiles via pinned `clang` | `wolio-ir`, `wolio-runtime`, `wolio-diagnostics` |
| `crates/wolio-agent` | `T211` | Supports `protocol_version: 2` (`wolio.mutation.v2`); adds `create_module`, `delete_module`, `add_import`, `remove_import` operations, server-side parsed canonical code strings (`code: string`), and V2 AST schemas; enforces `[architecture]` and visibility tier rules (`POST_REVIEW` vs `HUMAN_REQUIRED`) | `wolio-parser`, `wolio-semantic`, `wolio-verify`, `wolio-diagnostics` |
| `crates/wolio-format` | `T214` | Formats `predicate` declarations, `for..in..with..bound..invariant` loops, range expressions, turbofish calls, and `format(...)` calls canonically with 4-space LF indentation and `sym_` preservation check | `wolio-lexer`, `wolio-parser`, `wolio-ast`, `wolio-semantic`, `wolio-diagnostics` |
| `crates/wolio-lsp` | `T214` | Provides hover, go-to-definition, find-references, and workspace rename for `predicate` symbols, loop `item`/`acc` bindings, turbofish calls, and V2 builtin types | `wolio-parser`, `wolio-semantic`, `wolio-verify`, `wolio-format`, `wolio-diagnostics` |
| `crates/wolio-cli` | `T203`, `T216` | Dispatches `[language] version = "1" \| "2"` profile across `check`, `run`, `test`, `verify`, `inspect`, `impact`, `agent`, `build`, `fmt`, and `lsp` | All workspace service crates |

## 3. Core V2 representation and runtime ABI invariants

1. **Exact 128-bit fixed-scale `Decimal` (zero IEEE-754 floats):**
   - Represented in `wolio-runtime` and `wolio-ir` as a signed 128-bit integer (`i128`) storing the value scaled by `10^6` (`SCALE = 1_000_000`).
   - `Decimal::of(n: Int)` constructs `i128::from(n) * 1_000_000` (cannot overflow `i128` for `i64` inputs).
   - `Decimal::from_scaled(mantissa: Int, scale: Int)` requires `0 <= scale <= 6` and scales `mantissa` by `10^(6 - scale)`; `scale < 0` or `scale > 6` faults with `WOLIO-RUNTIME-001`.
   - Addition/subtraction use checked `i128` addition/subtraction.
   - **Standard multiplication and division trapping:** Standard `*` computes `(a * b)` and requires `p % 1_000_000 == 0`, faulting with `WOLIO-RUNTIME-001` on overflow or inexact fractional remainder; standard `/` computes `(a * 1_000_000) / b` and faults with `WOLIO-RUNTIME-001` on division by zero, overflow, or non-zero remainder (`(a * 1_000_000) % b != 0`), completely preventing silent precision loss.
   - **Explicit rounding operations:** `Decimal::mul_round(a, b, RoundingMode)` and `Decimal::div_round(a, b, RoundingMode)` apply explicit standard rounding modes (`HalfEven`, `HalfUp`, `Floor`, `Ceil`, `Truncate`). Division by zero faults with `WOLIO-RUNTIME-001`.
   - No `f32` or `f64` type or instruction is permitted anywhere in `wolio-*`.
2. **Checked bitwise operations on `Int` and `UInt`:**
   - Functional primitives: `bit_and`, `bit_or`, `bit_xor`, `bit_not`, `bit_shl`, `bit_shr`.
   - Both arguments for binary bitwise operations must have identical types (`Int` with `Int`, `UInt` with `UInt`).
   - Shift amounts must satisfy `0 <= n < 64`. Shift amounts `< 0` or `>= 64` fault immediately with `WOLIO-RUNTIME-001` (or `WOLIO-CONTRACT-004` inside contracts), preventing undefined behavior and silent wraparound.
3. **Value-semantic immutable `List<T>`, `Map<K, V>`, and `Set<T>` with turbofish constructors (ADR-009):**
   - Generic constructors are called with unambiguous turbofish syntax (`List::empty::<T>()`, `Map::empty::<K, V>()`, `Set::empty::<T>()`) or infer types from explicit binding annotations (`let l: List<Int> = List::empty();`).
   - In V2 (`M09`–`M11`), `List<T>`, `Map<K, V>`, and `Set<T>` are immutable reference-counted values (`Arc<Vec<Value>>`, `Arc<BTreeMap<MapKey, Value>>`, and `Arc<BTreeSet<MapKey>>` in the interpreter; opaque refcounted pointers in the C runtime ABI). Every `List::push`, `Map::insert`, `Map::remove`, `Set::insert`, and `Set::remove` returns a new collection without mutating existing aliases (in-place copy-on-write when `rc == 1` is deferred to V3 `T302`).
   - `Map::keys` and `Set::items` always yield elements in deterministic canonical order: signed numerical ascending order for `Int`, unsigned ascending order for `UInt`, `false < true` for `Bool`, and lexicographical UTF-8 byte order for `String` (unwrapping nominal wrappers to their underlying primitive order).
4. **Total bounded loop execution and IR verification:**
   - Every `for` loop in `VerifiedWolioIr` must lower to a structured loop region containing:
     1. Single evaluation of `iter_source` (`List<T>` length `N` or range `start..end` count `max(0, end - start)`),
     2. `BoundGuard` block evaluating `bound` once before iteration `0` and trapping with `WOLIO-CONTRACT-005` on `false`,
     3. Entry `InvariantGuard` block evaluating `invariant` with `acc = init` before iteration `0` and trapping with `WOLIO-CONTRACT-006` on `false`,
     4. Loop body advancing monotonic counter `0 <= idx < N` and computing `next_acc`,
     5. Step `InvariantGuard` block evaluating `invariant` with `acc = next_acc` after each iteration (`WOLIO-CONTRACT-006` on `false`), and
     6. Exit `InvariantGuard` verification before yielding final `acc`.
   - Arbitrary backward branches in `wolio-ir` outside a verified bounded loop header are rejected by `wolio_ir::verify_ir(...)`.
5. **Pure Rust LLVM codegen:**
   - Native compilation emits textual LLVM IR (`.ll`) and invokes pinned `clang` toolchain (zero Cargo `llvm-sys` / `inkwell` C++ build dependencies).
6. **Strict V1 isolation under `[language] version = "1"`:**
   - Running any V2 compiler binary on a project with `[language] version = "1"` must reject all V2 grammar/types (`WOLIO-FEATURE-001`) and produce bit-for-bit identical `sym_` declaration IDs, clause hashes, and snapshot hashes to V1.

