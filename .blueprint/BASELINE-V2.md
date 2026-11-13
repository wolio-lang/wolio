`.blueprint/BASELINE-V2.md`
# Wolio V2 — Bounded Iteration, Collections, Pure Predicates, and Architecture Policy

**Status:** planned post-V1 baseline, 2026-11-10. Design revision: `1.4` (`[language] version = "2"`).  
**Entry prerequisite:** Complete V1 (`M01`–`M07`) acceptance, or owner decision to advance after `M03`/`M04` trial + V1 Bug-Fix Gate (`M08`).

## 1. Purpose of V2

V1 proves the core thesis—nominal types, explicit `Result`/`Option`, runtime-guarded
contracts, explicit `effects`/`dependencies`, and structured AST mutations—on an
acyclic, loop-free language slice.

V2 completes the **core programming language** so Wolio can express real industrial
algorithms and multi-module domain architectures without opening the door to AI
workarounds:
1. **V1 Bug-Fix & Regression Hardening (`M08`):** Triage, reproduce with failing
   conformance fixtures, and fix all compiler, diagnostic, protocol, and backend
   defects discovered during V1 implementation and the M06 benchmark before adding
   any V2 syntax.
2. **Total Bounded Loops (`for ... in ... with ...`):** Give AI iteration over
   collections and integer ranges while making infinite loops (`while (true)`) and
   unverified loop state mutations impossible via mandatory `bound` and `invariant`
   clauses (`CORE_V2.md`).
3. **Immutable Value Collections & Exact Financial Math:** Add `List<T>`, `Map<K,V>`,
   `Decimal` (128-bit fixed-scale, zero IEEE-754 float rounding bugs), `UInt`,
   `Bytes`, and type-checked deterministic `format(...)`.
4. **Reusable Pure `predicate` Declarations:** Allow `requires`, `ensures`, `bound`,
   and `invariant` clauses to call reusable, guaranteed-pure, acyclic `predicate`
   declarations without duplicating complex boolean expressions.
5. **Multi-Module Agent Mutations & `[architecture]` Layer Enforcement:** Allow the
   agent protocol (`protocol_version: 2`) to create/delete modules and imports while
   statically enforcing module dependency allow/deny rules (`WOLIO-ARCH-001`) so AI
   can never import infrastructure from domain modules.

## 2. Non-collision guarantees with V1 and V3

- **Lexical compatibility:** Every V2 keyword (`for`, `in`, `with`, `bound`,
  `invariant`, `predicate`, `List`, `Map`, `Decimal`, `UInt`, `Bytes`, `format`) and
  every V3 keyword (`secret`, `Secret`, `resource`, `transaction`, `modifies`, `old`,
  `property`, `std`) is already reserved in `CORE_V1.md`. No valid V1 identifier can
  collide with V2 or V3 syntax.
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
| `M09` — Bounded Loops, Collections & Decimal | `T204`–`T208` | Parse and type-check `CORE_V2` (`for..in..with..bound..invariant`, `List<T>`, `Map<K,V>`, `Decimal`, `UInt`, `Bytes`, `format`); execute in interpreter with `bound`/`invariant` guards | Positive/negative loop bound, loop invariant, collection, and exact Decimal conformance suites |
| `M10` — Pure Predicates, Architecture Policy & Agent Protocol v2 | `T209`–`T212` | Implement `predicate` declarations and calls inside contracts, `[architecture]` module allow/deny rules (`WOLIO-ARCH-001`), and `protocol_version: 2` (`create_module`, `add_import`, loop/predicate AST payloads, loop-invariant analyzer rules) | Predicate cycle/purity rejection, architecture layer rejection, multi-module agent mutation fixtures |
| `M11` — V2 Native Parity, Tooling & Algorithmic Benchmark | `T213`–`T216` | Lower bounded loops, `List`/`Map`/`Decimal` to `VerifiedWolioIr` and LLVM; update `wolio fmt` and `wolio lsp` for V2; run 24-task algorithmic/collection benchmark (`W04`, `W07`, `W08`) | Interpreter/native differential parity on all V2 fixtures; published V2 benchmark report |

## 4. Normative V2 documents

- Language grammar and semantics: [CORE_V2.md](02-specifications/language/CORE_V2.md)
- Implementation architecture and crate extensions: [IMPLEMENTATION_SPECIFICATION_V2.md](03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Standalone task contracts (`T201`–`T216`): [task-index.md](04-tasks/task-index.md) and [V2-TASKS.md](04-tasks/V2-TASKS.md)

