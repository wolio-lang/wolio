`.blueprint/BASELINE-V2.md`
# Wolio V2 — Bounded Iteration, Collections, Pure Predicates, and Architecture Policy

**Status:** planned post-V1 baseline, 2026-11-14. Design revision: `1.5` (`[language] version = "2"`).  
**Entry prerequisite:** Complete V1 (`M01`–`M07`) acceptance, or owner decision to advance after `M03`/`M04` trial + V1 Bug-Fix Gate (`M08`).

## 1. Purpose of V2

V1 proves the core thesis—nominal types, explicit `Result`/`Option`, runtime-guarded
contracts, explicit `effects`/`dependencies`, and structured AST mutations—on an
acyclic, loop-free language slice.

V2 completes the **core systems & application programming language** so Wolio can
express real industrial algorithms, low-level data manipulation, and multi-module
domain architectures without opening the door to AI workarounds:
1. **V1 Bug-Fix & Regression Hardening (`M08`):** Triage, reproduce with failing
   conformance fixtures, and fix all compiler, diagnostic, protocol, and backend
   defects discovered during V1 implementation and the M06 benchmark before adding
   any V2 syntax.
2. **Total Bounded Loops (`for ... in ... with ...`):** Give AI iteration over
   collections and integer ranges while making infinite loops (`while (true)`) and
   unverified loop state mutations impossible via mandatory `bound` and `invariant`
   clauses (`CORE_V2.md`).
3. **Immutable Value Collections with Turbofish Syntax (ADR-009):** Add `List<T>`,
   canonical-order `Map<K, V>`, and canonical `Set<T>` with unambiguous generic turbofish
   constructor syntax (`List::empty::<T>()`, `Map::empty::<K, V>()`, `Set::empty::<T>()`)
   or declaration-target type inference, resolving Pratt parser `<` comparison ambiguities.
   Collections are pure, immutable, and content-addressed.
4. **Exact Financial Decimal Arithmetic & Explicit Rounding:** Add 128-bit fixed-scale
   `Decimal` (scale `10^6`, zero IEEE-754 float drift). Standard `*` and `/` operators
   strictly trap on unrepresentable fractional remainders, while explicit financial
   builtins `Decimal::mul_round(a, b, RoundingMode)` and `Decimal::div_round(a, b, RoundingMode)`
   provide audited rounding modes (`HalfEven`, `HalfUp`, `Floor`, `Ceil`, `Truncate`).
5. **Checked Bitwise Operations for Systems Programming:** Add functional bitwise
   primitives for `Int` and `UInt`: `bit_and`, `bit_or`, `bit_xor`, `bit_shl`, `bit_shr`,
   `bit_not`. Shift amounts are strictly controlled (`0 <= n < 64`) with zero silent
   wraparound or undefined behavior.
6. **Reusable Pure `predicate` Declarations:** Allow `requires`, `ensures`, `bound`,
   and `invariant` clauses to call reusable, guaranteed-pure, acyclic `predicate`
   declarations without duplicating complex boolean expressions.
7. **Multi-Module Agent Mutations & `[architecture]` Layer Enforcement:** Allow the
   agent protocol (`protocol_version: 2`) to create/delete modules and imports while
   statically enforcing module dependency allow/deny rules (`WOLIO-ARCH-001`) so AI
   can never import infrastructure from domain modules.
8. **Consolidated 14-Crate Toolchain & Pure Rust Codegen:** Implement all V2 compiler
   middle-end extensions inside `crates/wolio-semantic` (submodules `resolver`, `types`,
   `graph`, `contracts`, `effects`, `analyzer`), lower bounded loops to verified IR
   regions in `crates/wolio-ir`, and emit textual LLVM IR (`.ll`) compiled hermetically
   via pinned `clang` in `crates/wolio-codegen`.
9. **Full Algorithmic Comparative Benchmark Matrix (`T215`):** Execute a full 24-task
   × 10-seed × 4-language (960 runs) comparative evaluation across Wolio V2, Go, Rust,
   and TypeScript, evaluating loops, collections, Decimal, and architecture rules.

## 2. Non-collision guarantees with V1 and V3

- **Lexical compatibility:** Every V2 keyword (`for`, `in`, `with`, `bound`,
  `invariant`, `predicate`, `List`, `Map`, `Set`, `Decimal`, `UInt`, `Bytes`, `format`) and
  every V3 keyword (`secret`, `Secret`, `resource`, `transaction`, `modifies`, `old`,
  `property`, `std`, `with_mock`, `where`, `fn`, `foreign`) is already reserved in
  `CORE_V1.md` (24 forward-reserved keywords). No valid V1 identifier can collide
  with V2 or V3 syntax.
- **Manifest version gating:** Projects with `[language] version = "1"` continue to
  enforce V1 restrictions (`WOLIO-FEATURE-001` on V2 constructs); `[language] version = "2"`
  enables `CORE_V2.md`.
- **Additive diagnostics and IR:** V2 adds `WOLIO-CONTRACT-005` (loop bound failed),
  `WOLIO-CONTRACT-006` (loop invariant failed), and `WOLIO-ARCH-001` (forbidden
  module dependency edge) without altering any V1 diagnostic code meaning.

## 3. V2 delivery profiles (`M08`–`M11`)

| Milestone | Work units | Scope | Exit evidence |
|---|---|---|---|
| `M08` — V1 Bug-Fix & Regression Gate | `T201`–`T203` | Resolve all open V1 compiler/protocol/codegen defects; convert M06 benchmark failure patterns into regression tests; bump language gate for V2 | Zero open V1 P0/P1 bugs; all `BUG-*` fixtures pass in interpreter and native builds |
| `M09` — Bounded Loops, Collections, Bitwise & Decimal | `T204`–`T208` | Parse and type-check `CORE_V2` (`for..in..with..bound..invariant`, turbofish `List::*::<T>()`/`Map::*::<K,V>()`/`Set::*::<T>()`, checked bitwise ops, `Decimal` with explicit `RoundingMode`, `UInt`, `Bytes`, `format`); execute in interpreter with `bound`/`invariant` guards | Positive/negative loop bound, loop invariant, collection, bitwise, and exact Decimal conformance suites |
| `M10` — Pure Predicates, Architecture Policy & Agent Protocol v2 | `T209`–`T212` | Implement `predicate` declarations and calls inside contracts, `[architecture]` module allow/deny rules (`WOLIO-ARCH-001`), and `protocol_version: 2` (`create_module`, `add_import`, canonical code snippet strings / loop/predicate AST payloads, loop-invariant analyzer rules) | Predicate cycle/purity rejection, architecture layer rejection, multi-module agent mutation fixtures |
| `M11` — V2 Native Parity, Tooling & Algorithmic Benchmark | `T213`–`T216` | Lower bounded loops, collections, bitwise ops, and `Decimal` to `VerifiedWolioIr` and textual LLVM IR (`.ll`); update `wolio fmt` and `wolio lsp` for V2; run 24-task algorithmic/collection benchmark (`W04`, `W07`, `W08`, 960 runs) | Interpreter/native differential parity on all V2 fixtures; published 960-run V2 comparative benchmark report |

## 4. Normative V2 documents

- Paradigm authority: [ADR-009](01-architecture/decisions/ADR-009-unconventional-paradigm-unification.md)
- Language grammar and semantics: [CORE_V2.md](02-specifications/language/CORE_V2.md)
- Implementation architecture and crate extensions: [IMPLEMENTATION_SPECIFICATION_V2.md](03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Standalone task contracts (`T201`–`T216`): [task-index.md](04-tasks/task-index.md) and [V2-TASKS.md](04-tasks/V2-TASKS.md)

