`.blueprint/02-specifications/EXTENSION_BOUNDARIES.md`
# Version progression and extension boundaries (V1 -> V2 -> V3)

This document defines the strict boundary rules between V1 ([CORE_V1.md](language/CORE_V1.md)),
V2 ([CORE_V2.md](language/CORE_V2.md)), and V3 ([CORE_V3.md](language/CORE_V3.md)).

## 1. Upward compatibility rules

1. **Reserved lexical space:** Every V2 and V3 keyword is reserved in the V1
   lexer (`CORE_V1.md`). A valid V1 program can never use an identifier that
   later becomes a keyword in V2 or V3.
2. **Manifest version gating:** `[language] version = "1" | "2" | "3"` in
   `wolio.toml` selects the active grammar and semantic profile. Using a V2
   construct under `version = "1"` or a V3 construct under `version = "1"`/`"2"`
   fails deterministically with `WOLIO-FEATURE-001`.
3. **Bug-fix before feature expansion:** Neither V2 (`M09`) nor V3 (`M13`) may
   begin feature implementation until the preceding version's dedicated Bug-Fix
   & Regression Gate (`M08` for V1->V2, `M12` for V2->V3) has closed all open
   P0/P1 defects with permanent conformance regression fixtures.

## 2. Capability progression matrix

| Capability | V1 (`version = "1"`) | V2 (`version = "2"`) | V3 (`version = "3"`) |
|---|---|---|---|
| Control flow | `if`/`else`, exhaustive `match`, acyclic calls | + Total bounded `for (x in src with acc = init) { bound ...; invariant ...; }` | Same as V2 (no unbounded `while`) |
| Builtin types | `Bool`, `Int`, `String`, `Unit`, `Option<T>`, `Result<T,E>` | + `UInt`, `Decimal` (128-bit fixed scale, explicit `RoundingMode`), checked bitwise ops (`bit_and`, `bit_or`, `bit_xor`, `bit_shl`, `bit_shr`, `bit_not`), `Bytes`, `List<T>`, `Map<K,V>` (turbofish `::<...>`) | + `Secret<T>`, linear `resource <Handle>`, linear opaque `foreign type`, fixed-width integers (`UInt8`..`UInt32`, `Int8`..`Int32`), `std::*` types |
| String formatting | Literal strings only | + Type-checked deterministic `format(template, ...)` | + `std::string` pure manipulation & parsing (`Secret<T>` rejected) |
| Contract clauses | Inline `requires`/`ensures` (no calls), `errors`, `effects`, `dependencies`, `intent` | + Reusable pure acyclic `predicate` calls in `requires`/`ensures`/`bound`/`invariant` | + `modifies [res]`, `old(expr)` snapshots, `foreign` operation pre/post C boundaries, and `property` fuzzing |
| Architecture rules | Explicit operation `dependencies` and module `import` | + `[architecture]` allow/deny module import rules (`WOLIO-ARCH-001`) | Same as V2 + `std::*` & FFI capability boundaries (`[foreign].allowed_libraries`) |
| Agent mutations | Single/existing-module AST declarations (`protocol_version: 1`), canonical code snippet strings | + `create_module`, `delete_module`, `add_import`, `remove_import` (`protocol_version: 2`), canonical code snippet strings | + `property`, `resource`, `foreign`, fixed-width integers, and `with_mock` AST payloads (`protocol_version: 3`) |
| External I/O & tests | `console::print` (`Console::Write`) only; captured in tests | Same as V1 | + `Clock`, `Fs`, `Net`, `Db`, `Secret::Use`, `Foreign::Call`; mandatory `with_mock` in tests (`WOLIO-EFFECT-002`) |
| Verification depth | Static type/set checks + runtime guards + unit `test` + `WOLIO-ANALYZER-001`..`005` | + Loop `bound`/`invariant` checks (`WOLIO-CONTRACT-005`/`006`) + architecture check | + Automatic contract/property fuzzing (`wolio verify --fuzz`, `WOLIO-FUZZ-001`) |
