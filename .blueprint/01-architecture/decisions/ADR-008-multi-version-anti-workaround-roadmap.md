`.blueprint/01-architecture/decisions/ADR-008-multi-version-anti-workaround-roadmap.md`
# ADR-008 — Industrial anti-workaround rules, self-contained tasks, and non-colliding V1/V2/V3 architecture

**Status:** Proposed for owner acceptance with blueprint revision 1.4  
**Date:** 2026-11-10

## Context

Wolio is an industrial-grade programming language designed specifically for AI software construction. Its primary engineering goal is to prevent AI reward-hacking and workarounds ("akal-akalan")—such as swallowing errors, hardcoding test outputs, bypassing contracts, introducing infinite loops, or creating hidden architectural dependencies—even at the cost of higher source verbosity and lower manual coding convenience for humans.

Audit of Revision 1.3 identified four structural issues before freezing the blueprint:
1. **Conflicting historical files:** 46 superseded draft files (`LANGUAGE_SPEC.md`, `SCOPE.md`, `WTP_SPEC.md`, 18 crate sketches in `03-implementation/crates/`, 7 milestone drafts in `03-implementation/milestones/`, and `M00-*` reports) remained in `.blueprint/` with incompatible syntax, error codes (`LEX001`), and milestone rules, creating context pollution for coding agents.
2. **Unresolved V1 task blockers and analyzer loopholes:** `T020` had an open question on cycle diagnostic codes; `T024` had an open question on which contract clauses trigger `WOLIO-FEATURE-001` at M02; `WOLIO-ANALYZER-002` and `003` were unemitted aliases; and `WOLIO-ANALYZER-004` checked only raw AST literals (allowing `99 + 1` to bypass a forbidden `100`).
3. **Cross-file reading overhead:** Task contracts relied on external lookups for crate layouts, keyword sets, and error mappings instead of embedding a self-contained implementation reference.
4. **Collision risk across V1, V2, and V3:** Deferring loops, collections, pure predicates, and the standard library to V2 and V3 without reserving their lexical keywords and version boundaries in V1 risked breaking V1 programs when upgrading to V2 or V3.

## Decision

1. **Purge Superseded Historical Files:** Remove all 46 superseded draft files and empty directories from `.blueprint/`. Every file remaining in `.blueprint/` is active and authoritative for its designated version (`V1`, `V2`, or `V3`).
2. **Forward-Reserve V2 and V3 Keywords in V1:** The V1 lexer (`CORE_V1.md`, `INTERFACE_CONTRACTS.md`, `T002`) reserves all V2 (`in`, `with`, `bound`, `invariant`, `predicate`, `List`, `Map`, `Decimal`, `UInt`, `Bytes`, `format`) and V3 (`secret`, `Secret`, `resource`, `transaction`, `modifies`, `old`, `property`, `std`) identifiers as reserved keywords. Using a V2/V3 keyword as an identifier in V1 fails lexing/parsing (`WOLIO-SYNTAX-001`); using a V2/V3 language construct under `[language] version = "1"` fails with `WOLIO-FEATURE-001`.
3. **Resolve All V1 Task Ambiguities:**
   - Import cycles and recursive call cycles in `T020` emit `WOLIO-RESOLVE-001` with structured cycle path details.
   - At `M02` (`T024`), `effects`, `dependencies`, `errors`, and `intent` clauses are parsed and permitted (allowing `console::print` declarations to run in M02), while static verification of those sets is reported `not_run` until `M03` (`T031`). Only `requires` and `ensures` predicate guards trigger `WOLIO-FEATURE-001` when `wolio run` is invoked at `M02` before `T030`.
4. **Harden V1 Anti-Workaround Analyzer (`WOLIO-ANALYZER-001`..`005`):**
   - `WOLIO-ANALYZER-001` (error): Rejects any mutation request or annotation attempting diagnostic suppression or verification bypass.
   - `WOLIO-ANALYZER-002` (error): Rejects suspicious error-swallowing substitution where an operation returning `Result` matches a callee `Result` and its `Result::Err(_)` arm discards the error payload to return a constant `Result::Ok(...)`.
   - `WOLIO-ANALYZER-003` (error): Rejects declared bloat where an operation declares a symbol in `dependencies` or an effect in `effects` that is neither called nor transitively produced by its implementation body.
   - `WOLIO-ANALYZER-004` (error): Rejects configured `forbidden_int_literals` and `forbidden_string_literals` across both raw AST literals and **constant-folded pure expressions** (e.g., `99 + 1`) in production implementations.
   - `WOLIO-ANALYZER-005` (warning): Emits a warning on structurally identical normalized `match` arms OR **test-literal mirroring** (where a production `if`/`match` condition compares a parameter directly against an exact non-boundary literal input used in a `test` block for that operation).
5. **Non-Colliding 3-Version Industrial Roadmap (`V1 -> V2 -> V3`):**
   - **V1 (`BASELINE-V1.md`, `CORE_V1.md`, `M01`–`M07`, `T001`–`T074`):** Immutable acyclic core, nominal types, exhaustive `Result`/`Option`, runtime-guarded contracts, deterministic `sym_` graph, local JSON-AST agent protocol, verified IR + LLVM backend, 24-task benchmark, and Git package manager / formatter / LSP.
   - **V2 (`BASELINE-V2.md`, `CORE_V2.md`, `M08`–`M11`, `T201`–`T216`):** Preceded by the V1 Bug-Fix & Regression Gate (`M08`). Adds total bounded loops (`for ... in ... with ... { bound ...; invariant ...; ... }`), immutable `List<T>` and `Map<K,V>`, exact `Decimal`/`UInt`/`Bytes`, safe deterministic `format(...)`, pure reusable `predicate` declarations, multi-module agent mutations (`create_module`, `add_import`), and `[architecture]` layer allow/deny enforcement.
   - **V3 (`BASELINE-V3.md`, `CORE_V3.md`, `M12`–`M15`, `T301`–`T315`):** Preceded by the V2 Bug-Fix & Regression Gate (`M12`). Adds industrial `std::*` modules, capability-gated external effects with mandatory deterministic test mocks, `Secret<T>` redaction and validation provenance, stateful `resource`/`transaction` frames, and contract-driven property fuzzing (`property` / `wolio verify --fuzz`).
6. **Self-Contained Task Contracts:** Every task file embeds its exact crate boundaries, required data structures/rules, diagnostic codes, and verification commands so an agent executing a single task does not need to read the entire repository.

## Consequences

- Zero conflicting historical documents remain in `.blueprint/`.
- Every V1 program (`[language] version = "1"`) is upward-compatible with V2 and V3 compilers without identifier collisions.
- AI workarounds (constant-folding bypass, error-to-Ok swallowing, dependency padding, test-input hardcoding, and infinite loops) are blocked by construction across V1, V2, and V3.

