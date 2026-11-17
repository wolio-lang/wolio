`.blueprint/BASELINE-V1.md`
# Wolio V1 — Freeze Baseline

**Status:** freeze candidate, 2026-11-14. Design revision: `1.5`.

Read [DOCUMENT_AUTHORITY.md](DOCUMENT_AUTHORITY.md) for the complete active-document
register and [PROJECT_COVERAGE.md](00-product/PROJECT_COVERAGE.md) for how V1
(`BASELINE-V1.md`), V2 ([BASELINE-V2.md](BASELINE-V2.md)), and V3
([BASELINE-V3.md](BASELINE-V3.md)) fit together without collision.

This document defines the complete V1 product scope. Implementation is not implied
by design completion. The project owner records the actual freeze in
`.execution/baseline-acceptance.md` after reviewing this revision.

## Product decision

Wolio is an industrial-grade, contract-first programming language built specifically
for **Industrial Vibe Coding**—where the human acts as the system architect/specifier
and never needs to read generated code, while the machine generates and verifies all
implementations. Its primary engineering goal is to kill bugs from the very start and
eliminate AI reward-hacking ("akal-akalan")—such as swallowed errors, hardcoded
test-passing outputs, undeclared dependencies/effects, and silent contract weakening.

Wolio enforces an absolute **Zero-Unsafe Policy**: there is no `unsafe` keyword, no
raw pointers, no manual memory management, and no in-source compiler suppression
directives (`#[allow]`, `nolint`). Wolio deliberately trades away manual human coding
brevity and readability in favor of 100% mechanical verifiability: verbose explicit
declarations (`dependencies [...]`, `effects [...]`, `errors [...]`, nominal wrappers,
and multi-tiered analyzer constraints) are deliberate machine ratchets that make
defects impossible to compile.

## Delivery profiles (V1: M01–M07)

| Profile | Delivery | Work units | Included | Exit evidence |
|---|---|---|---|---|
| Frontend | M01 | T001–T004 | init, lexer (with V1+V2+V3 reserved keywords), parser, AST, JSON diagnostics | valid/invalid grammar fixtures |
| Executable core | M02 | T020–T024 | resolution, cycle rejection, nominal/record/enum types, Result/Option, exhaustive match, `sym_` IDs, interpreter, `run` | executable examples and negative type/cycle fixtures |
| Trial | M03 | T030–T034 | `requires`/`ensures` guards, static error/effect/dependency sets, `test`/`verify`, `inspect`/`impact`, pilot benchmark | contract failures, three change cycles, raw pilot results |
| Agent | M04 | T040–T044 | local JSONL protocol, typed AST mutation plans, scope/approvals, CAS/journal recovery, `WOLIO-ANALYZER-001`..`005` | stale/rejected/accepted mutation and anti-workaround fixtures |
| Native | M05 | T050–T053 | typed IR, `VerifiedWolioIr` verifier, LLVM lowering, C/runtime ABI, interpreter/native parity | interpreter/native differential corpus |
| Evaluation | M06 | T060–T063 | 10-task contract-first benchmark across Wolio, Go, Rust, TypeScript (200 runs; 24-task algorithmic benchmark in V2 T215) | reproducible bootstrap report, including negative findings |
| Toolchain | M07 | T070–T074 | Git packages, SHA-256 cache/offline lock, canonical formatter, LSP, Linux x86_64 archive | offline locked build, format idempotence, editor diagnostics |

`M03` is a legitimate standalone trial checkpoint. `M04` and `M05` independently
depend on `M03`. `M06` requires `M04`. `M07` depends on `M04` and `M05`. Full V1
ends at `M07`. Loops, collections (`List`, `Map`), `Decimal`, pure `predicate`
helpers, and `[architecture]` module policies belong to [BASELINE-V2.md](BASELINE-V2.md);
the standard library (`std::*`), capability-gated I/O mocks, `Secret<T>`,
transactions, and contract fuzzing belong to [BASELINE-V3.md](BASELINE-V3.md).

## Normative reading map

- Concrete V1 syntax, forward-reserved V2/V3 keywords, and execution semantics:
  [CORE_V1.md](02-specifications/language/CORE_V1.md).
- Behavioral obligations and compatibility:
  [CONTRACT_DESIGN.md](02-specifications/CONTRACT_DESIGN.md).
- Deterministic identity, semantic graph, and provenance:
  [SEMANTIC_MODEL.md](02-specifications/SEMANTIC_MODEL.md).
- Shared project layout, CLI envelopes, canonical hashes, and lexical rules:
  [INTERFACE_CONTRACTS.md](02-specifications/INTERFACE_CONTRACTS.md).
- Diagnostics, anti-workaround analyzer rules (`001`–`005`), and evidence model:
  [DIAGNOSTIC_SPEC.md](02-specifications/verification/DIAGNOSTIC_SPEC.md),
  [EVIDENCE_MODEL.md](02-specifications/verification/EVIDENCE_MODEL.md).
- Local agent protocol, mutation schema, and risk tiers:
  [LOCAL_PROTOCOL_V1.md](02-specifications/protocol/LOCAL_PROTOCOL_V1.md),
  [MUTATION_SCHEMA_V1.md](02-specifications/protocol/MUTATION_SCHEMA_V1.md),
  [APPROVAL_MATRIX_V1.md](02-specifications/protocol/APPROVAL_MATRIX_V1.md).
- Package manager:
  [PACKAGE_MANAGER_SPEC.md](02-specifications/package-manager/PACKAGE_MANAGER_SPEC.md).
- Standalone task contracts (`T001`–`T074`):
  [task-index.md](04-tasks/task-index.md) and [V1-TASKS.md](04-tasks/V1-TASKS.md).
- Toolchain, architecture edges, and delivery gates:
  [IMPLEMENTATION_SPECIFICATION_V1.md](03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md),
  [DELIVERY_POLICY.md](03-implementation/DELIVERY_POLICY.md),
  [DELIVERY_PLAN.md](06-project-management/DELIVERY_PLAN.md),
  [ACCEPTANCE_MATRIX.md](05-quality/ACCEPTANCE_MATRIX.md).

## Revision 1.5 decisions

Revision 1.5 incorporates critical architecture and vibe-coding optimizations
to ensure Wolio serves as a robust systems language for large-scale enterprise
construction without scope bloat, guided by [ADR-009](01-architecture/decisions/ADR-009-unconventional-paradigm-unification.md),
[ADR-008](01-architecture/decisions/ADR-008-multi-version-anti-workaround-roadmap.md),
and [ADR-007](01-architecture/decisions/ADR-007-deterministic-identity-and-gated-automation.md):
1. **Keyword Forward-Reservation:** Expanded Set 3 forward-reserved keywords to
   24 words by adding `Set`, `with_mock`, `where`, `fn`, and `foreign` (for Systems FFI),
   guaranteeing zero source collisions across V1, V2, and V3.
2. **LLM Mutation Ergonomics:** Solved the JSON-AST verbosity paradox in
   `MUTATION_SCHEMA_V1` and `LOCAL_PROTOCOL_V1` by permitting canonical Wolio
   source code strings (`code: string`) in mutation payloads, parsed server-side
   into verified AST diffs while preserving 100% strict semantic bounds.
3. **Early Anti-Hardcoding Defense:** Added automated scalar boundary and property
   fuzzing (`wolio verify --fuzz`) with `WOLIO-VERIFY-002` in M03 (`T032`),
   defeating AI test-overfitting on scalar types directly in V1.
4. **Rescoped V1 Benchmark:** Refined M06 from a 960-run matrix into a 10-task
   Contract & Protocol Benchmark (200 runs) evaluating contract verification,
   error handling, and agent repair vs Go, Rust, TS; the 24-task algorithmic
   benchmark matrix is formally scheduled for V2 M11 (`T215`).
5. **Crate Consolidation:** Consolidated the compiler middle-end into
   `crates/wolio-semantic` (submodules `resolver`, `types`, `graph`, `contracts`,
   `effects`, `analyzer`), reducing workspace crate count from 19 to 14 and
   eliminating cyclic dependency hazards.
6. **Hermetic Native Lowering:** Pinned native codegen in `wolio-codegen` to
   verified textual LLVM IR (`.ll`) emission and `clang` compilation, eliminating
   brittle Cargo C++ FFI build risks (`llvm-sys`/`inkwell`).
7. **Conformance Fixture Header Standard:** Specified the standard machine-readable
   test header format (`// @mode:`, `// @expect-status:`, `// @expect-code:`) across
   `tests/conformance/`.
8. **Compacted Systems Roadmap:** Aligned the full systems capabilities across V1
   (scalar & IR foundation), V2 (bitwise ops, exact math), and V3 (capability-gated
   FFI & fixed-width integers), strictly avoiding a separate V4.

## Freeze and amendment

Freeze fixes product behavior, feature boundaries, acceptance cases, and schema
versions. An amendment after owner freeze requires an ADR containing old/new
rules, rationale, compatibility, affected fixtures/tasks, and owner decision.
