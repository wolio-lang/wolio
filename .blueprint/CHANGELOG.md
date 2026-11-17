`.blueprint/CHANGELOG.md`
# Blueprint Changelog

> Non-normative history of blueprint revisions. Active rules are defined by
> [DOCUMENT_AUTHORITY.md](DOCUMENT_AUTHORITY.md) and its binding documents.

## Unreleased

### Revision 1.5 — 2026-11-14 (current freeze candidate)

- **Vibe-Native Zero-Inspection Architecture Overhaul (ADR-010):**
  - Codified the definitive Vibe-Native Zero-Code Inspection architecture across Seven Foundational Pillars:
    1. Adversarial Dual-Agent Property Synthesis (Generator vs. Critic) replacing manual boolean contracts.
    2. Macro-Capability Envelopes in `wolio.toml` replacing micro-AST approvals (`AUTO` internal refactoring).
    3. MCTS Search-Tree & Deterministic Auto-Fix with state rollback replacing 3-attempt linear abort.
    4. Structural Termination & Vectorized Streams (`map`, `filter`, `fold`, etc.) replacing manual loop invariants.
    5. Sandboxed WASI Component Model FFI (WIT interfaces, isolated memory) replacing brittle C-glue FFI.
    6. Schema-Driven Property Fuzzing and Ephemeral Container Live Sandboxes replacing hallucinated `with_mock`.
    7. Actor Model & Supervision Trees (process-isolated state, typed mailboxes, restart trees) replacing static-only DAG concurrency.
  - Codified Four Anti-Pathology Mechanical Safeguards against multi-session "script akal-akalan":
    1. Semantic Context Digest (`.wolio/context-spine.json` ~2,000 tokens) providing 100% global architecture awareness without reading function bodies.
    2. Zero-Tolerance Anti-Orphan Architecture rejecting unmapped scripts (`fix_temp.py`) at Tier 0 syntax/module resolution (`WOLIO-MODULE-001`).
    3. Automated Global Regression Gate emitting Semantic Root-Cause Diagnostics.
    4. Nominal Refinement Types preventing fake data or string sentinel workarounds.
- **Agent Autonomy & Approval Protocol Overhaul (Session 2):**
  - **Macro-Capability Approval Matrix (`APPROVAL_MATRIX_V1.md`):** Eliminated micro-AST approvals. All internal codebase evolutions (private declarations, internal operations, internal errors, internal effects/I/O within declared macro-capabilities, private refactoring, and auto-pruning) are classified as `AUTO`. Human review (`HUMAN_REQUIRED`) is restricted strictly and exclusively to: (1) External capability escalations (new network domains, filesystem paths outside sandbox roots), (2) Breaking public API schema changes (deleting public symbols, breaking signatures, weakening public contracts), and (3) Runtime/computational resource budget violations (`timeout_ms`, `max_memory_mb`). `POST_REVIEW` is retained strictly for non-breaking, additive public boundary changes.
  - **Context Spine Generator (`LOCAL_PROTOCOL_V1.md`, `MUTATION_SCHEMA_V1.md`):** Formalized compiler-generated architectural index at `.wolio/context-spine.json` strictly budgeted to ≤ 2,000 tokens. In multi-session agent workflows (Sessions 4, 10+), AI agents consume this spine for 100% global architectural visibility, permanently defeating Context Window Amnesia and preventing duplicate domain types (`WOLIO-TYPE-002`) and unmapped orphan scripts (`WOLIO-MODULE-001`).
  - **Mechanical Auto-Prune Pass:** Reclassified declared bloat (unused dependencies, unreferenced imports, dead bindings) from fatal compilation rejections into a deterministic compiler AST rewrite pass, logging pruned entities under `auto_pruned: [...]` without failing plans or triggering panic workarounds.
  - **MCTS Search-Tree & State Rollback:** Permanently abolished the legacy "3-attempt bounded linear abort". Implemented Monte Carlo Tree Search exploration of mutation branches with clean, transactional state rollbacks to valid checkpoints on failure, outputting semantic root-cause diagnostic trees upon exploration budget exhaustion.
  - **Task & Spec Harmonization:** Synchronized `T040`, `T042`, `T043`, `T044`, `V1-TASKS.md`, `PROJECT_COVERAGE.md`, `DELIVERY_PLAN.md`, `ACCEPTANCE_MATRIX.md`, `IMPLEMENTATION_SPECIFICATION_V1.md`, and `AI_BUG_TAXONOMY.md`.
- Addressed industrial vibe-coding and systems architecture review:
  - **Unconventional Paradigm Unification (ADR-009):** Codified Wolio's radical architectural departure from conventional languages across four immutable pillars: (1) Zero-coloring deterministic dataflow parallelism replacing threads and async/await; (2) Canonical content-addressed value collections (`Set<T>`, `Map<K, V>`) and relational identity graphs over `sym_id` tuples replacing mutable pointer heaps; (3) Contractual refinement and exhaustive closed sums replacing class inheritance and open trait hierarchies; (4) Linear resource frames, atomic rollbacks, and mandatory hermetic test isolation (`with_mock`) replacing mutable pointer I/O.
  - **Canonical Set Collection:** Added `Set<T>` to V2 M09 (`CORE_V2.md`, `T207`, `T213`, `BASELINE-V2.md`) and forward-reserved `Set` in V1 Set 3 keywords (expanding to 24 forward-reserved keywords).
  - **Forward-Reserved Keywords:** Added `Set`, `with_mock`, `where`, `fn`, and `foreign` to V1 Set 3 forward-reserved keywords (24 total keywords) across `CORE_V1.md`, `INTERFACE_CONTRACTS.md`, and `T002`.
  - **LLM Mutation Ergonomics:** Solved JSON-AST verbosity paradox in `MUTATION_SCHEMA_V1.md` and `LOCAL_PROTOCOL_V1.md` by allowing canonical Wolio `code: string` payloads, parsed server-side into typed AST diffs by `wolio-parser`.
  - **Early Anti-Hardcoding Defense:** Added scalar boundary and property fuzzing in M03 (`wolio verify --fuzz`) with `WOLIO-VERIFY-002` to defeat test-overfitting on scalar types directly in V1.
  - **M06 Rescoping:** Refined V1 M06 evaluation from a 960-run algorithmic matrix to a 10-task Contract & Protocol Benchmark (200 runs), reserving the 24-task algorithmic benchmark for V2 M11 (`T215`).
  - **Crate Consolidation:** Merged compiler middle-end into `crates/wolio-semantic` (submodules `resolver`, `types`, `graph`, `contracts`, `effects`, `analyzer`), reducing workspace crates from 19 to 14 and eliminating inverted dependency hazards.
  - **Pure Rust Native Codegen:** Pinned `wolio-codegen` to verified textual LLVM IR (`.ll`) emission and `clang` compilation, eliminating brittle Cargo `llvm-sys` / `inkwell` C++ build risks.
  - **Conformance Fixture Header Specification:** Standardized test execution directives (`// @mode:`, `// @expect-status:`, `// @expect-code:`) for all `tests/conformance/` fixtures.
  - **Single Normative Authority:** Designate `04-tasks/Tnnn-*.md` as the sole normative source for all tasks, eliminating dual-source hazards with summary indexes.
  - **Compact Systems Strategy:** Incorporated full systems roadmap across V1 (scalar & IR foundation), V2 (bitwise ops, exact math), and V3 (capability-gated FFI & fixed-width integers), strictly avoiding a separate V4.
  - **V2 Turbofish Syntax:** Resolved Pratt parser `<` comparison ambiguity by establishing explicit turbofish syntax (`List::empty::<T>()`, `Map::empty::<K, V>()`) and declaration-target type inference.
  - **V2 Checked Bitwise Operations:** Added functional checked bitwise operations (`bit_and`, `bit_or`, `bit_xor`, `bit_shl`, `bit_shr`, `bit_not`) for `Int` and `UInt` with strictly bounded shift amounts (`0 <= n < 64`).
  - **V2 Exact Decimal with Explicit Rounding:** Enforced strict trapping on inexact unscaled remainders for standard `*` and `/`, while providing explicit audited rounding via `Decimal::mul_round` and `Decimal::div_round` with `RoundingMode` enum (`HalfEven`, `HalfUp`, `Floor`, `Ceil`, `Truncate`).
  - **V2 14-Crate Architectural Alignment:** Synchronized `IMPLEMENTATION_SPECIFICATION_V2.md` and all V2 task contracts (`T201`–`T216`) to the consolidated 14-crate DAG (`crates/wolio-semantic` middle-end) and pure Rust textual LLVM IR emission.
  - **V2 Full Algorithmic Benchmark Allocation:** Pinned the 24-task × 10-seed × 4-language (960 runs) comparative algorithmic benchmark to M11 (`T215`).
  - **V3 Capability-Gated Foreign Function Interface (FFI):** Added `foreign type` (linear, opaque, non-storable `WOLIO-FOREIGN-002`, scope auto-drop glue) and `foreign operation` (contract-guarded interop via compiler-generated C glue) with manifest gating in `wolio.toml [foreign].allowed_libraries` (`WOLIO-FOREIGN-001`, `HUMAN_REQUIRED`) and mandatory test mocking (`Foreign::Call`, `WOLIO-EFFECT-002`).
  - **V3 Fixed-Width Integer Scalar Types:** Added `UInt8`..`UInt32` and `Int8`..`Int32` with checked arithmetic (`WOLIO-RUNTIME-001`), explicit checked conversions (`UInt8::from_uint`), zero silent truncation, and native LLVM IR mapping (`i8`, `i16`, `i32`).
  - **V3 Linear Resource Instantiation Deadlock Resolution:** Defined scope-bound acquisition `Resource::acquire::<T>(...)` at entrypoints and `Resource::mock::<T>()` in `with_mock` test blocks, with strict non-aliasing (`WOLIO-RESOURCE-001`), `modifies` frames, and `old(...)` pre-state snapshots.
  - **V3 14-Crate Alignment & Task Expansion:** Synchronized `IMPLEMENTATION_SPECIFICATION_V3.md` and all V3 tasks to the 14-crate DAG (`crates/wolio-semantic` middle-end) and pure Rust textual LLVM IR emission; added tasks `T316`–`T319` in Milestone M14 for FFI and fixed-width types, completing V3 with 19 self-contained tasks (`T301`–`T319`).

### Revision 1.4 — 2026-11-10 (superseded by revision 1.5)

- Incorporated [ADR-008](01-architecture/decisions/ADR-008-multi-version-anti-workaround-roadmap.md):
  - Removed all 46 superseded historical draft files (`LANGUAGE_SPEC.md`, `SCOPE.md`, `WTP_SPEC.md`, 18 crate sketches in `03-implementation/crates/`, 7 milestone drafts in `03-implementation/milestones/`, and `M00-*` reports) to eliminate conflicting syntax and error codes.
  - Reserved all V2 and V3 keywords in the V1 lexer (`CORE_V1.md`, `INTERFACE_CONTRACTS.md`, `T002`) to guarantee zero source collision across V1, V2, and V3.
  - Resolved all remaining task `Open:` items in `T020` (`WOLIO-RESOLVE-001` for import/call cycles) and `T024` (declarative `effects`/`dependencies`/`errors`/`intent` permitted in M02; only `requires`/`ensures` trigger `WOLIO-FEATURE-001` at M02 `run`).
  - Strengthened `WOLIO-ANALYZER-001`..`005` in `DIAGNOSTIC_SPEC.md` and `T044` so `002` (swallowed error returning constant `Ok`), `003` (unused declared dependency/effect padding), `004` (constant-folded forbidden literals), and `005` (identical arms and test-literal mirroring) are concrete, enforceable checks.
  - Added complete, non-colliding blueprints for **V2** ([BASELINE-V2.md](BASELINE-V2.md), [CORE_V2.md](02-specifications/language/CORE_V2.md), [V2-TASKS.md](04-tasks/V2-TASKS.md): `M08`–`M11`, `T201`–`T216`) and **V3** ([BASELINE-V3.md](BASELINE-V3.md), [CORE_V3.md](02-specifications/language/CORE_V3.md), [V3-TASKS.md](04-tasks/V3-TASKS.md): `M12`–`M15`, `T301`–`T315`), including mandatory inter-version Bug-Fix & Regression Gates (`M08` and `M12`).
  - Made task files self-contained and aligned `.execution/baseline-acceptance.md` to Revision 1.4.

### Revision 1.3 — 2026-11-06 (superseded by revision 1.4)

- Incorporated [ADR-007](01-architecture/decisions/ADR-007-deterministic-identity-and-gated-automation.md): removed `wolio.ids.json`, defined deterministic `sym_` SHA-256 declaration identities, required typed-only AST agent mutations, introduced `AUTO`/`POST_REVIEW`/`HUMAN_REQUIRED` risk tiers, made the interpreter the default development backend, and restricted LLVM to `VerifiedWolioIr`.

### Revision 1.2 — 2026-11-06 (superseded by revision 1.3)

- Added authoritative document register, whole-project coverage, shared CLI/manifest/hash contracts, and expanded V1 task contracts `T020`–`T074`.
