`.blueprint/BASELINE-V1.md`
# Wolio V1 — Freeze Baseline

**Status:** freeze candidate, 2026-11-10. Design revision: `1.4`.

Read [DOCUMENT_AUTHORITY.md](DOCUMENT_AUTHORITY.md) for the complete active-document
register and [PROJECT_COVERAGE.md](00-product/PROJECT_COVERAGE.md) for how V1
(`BASELINE-V1.md`), V2 ([BASELINE-V2.md](BASELINE-V2.md)), and V3
([BASELINE-V3.md](BASELINE-V3.md)) fit together without collision.

This document defines the complete V1 product scope. Implementation is not implied
by design completion. The project owner records the actual freeze in
`.execution/baseline-acceptance.md` after reviewing this revision.

## Product decision

Wolio is an industrial-grade, contract-first programming language built specifically
for AI-assisted software construction. Its primary engineering goal is to eliminate
AI reward-hacking and workarounds ("akal-akalan")—such as swallowed errors,
hardcoded test-passing outputs, undeclared dependencies/effects, and silent contract
weakening—that turn fast AI coding into long-term technical debt.

Wolio deliberately trades away manual human coding brevity in favor of machine
verifiability: verbose explicit declarations (`dependencies [...]`, `effects [...]`,
`errors [...]`, nominal wrappers, and typed AST mutations) are accepted costs on
small programs because they enforce deterministic, low-defect scaling on large
industrial systems.

## Delivery profiles (V1: M01–M07)

| Profile | Delivery | Work units | Included | Exit evidence |
|---|---|---|---|---|
| Frontend | M01 | T001–T004 | init, lexer (with V1+V2+V3 reserved keywords), parser, AST, JSON diagnostics | valid/invalid grammar fixtures |
| Executable core | M02 | T020–T024 | resolution, cycle rejection, nominal/record/enum types, Result/Option, exhaustive match, `sym_` IDs, interpreter, `run` | executable examples and negative type/cycle fixtures |
| Trial | M03 | T030–T034 | `requires`/`ensures` guards, static error/effect/dependency sets, `test`/`verify`, `inspect`/`impact`, pilot benchmark | contract failures, three change cycles, raw pilot results |
| Agent | M04 | T040–T044 | local JSONL protocol, typed AST mutation plans, scope/approvals, CAS/journal recovery, `WOLIO-ANALYZER-001`..`005` | stale/rejected/accepted mutation and anti-workaround fixtures |
| Native | M05 | T050–T053 | typed IR, `VerifiedWolioIr` verifier, LLVM lowering, C/runtime ABI, interpreter/native parity | interpreter/native differential corpus |
| Evaluation | M06 | T060–T063 | 24-task × 10-seed paired benchmark across Wolio, Go, Rust, TypeScript | reproducible bootstrap report, including negative findings |
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
- Toolchain, architecture edges, and delivery gates:
  [IMPLEMENTATION_SPECIFICATION_V1.md](03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md),
  [DELIVERY_POLICY.md](03-implementation/DELIVERY_POLICY.md),
  [DELIVERY_PLAN.md](06-project-management/DELIVERY_PLAN.md),
  [ACCEPTANCE_MATRIX.md](05-quality/ACCEPTANCE_MATRIX.md).

## Revision 1.4 decisions

[ADR-008](01-architecture/decisions/ADR-008-multi-version-anti-workaround-roadmap.md)
builds on [ADR-007](01-architecture/decisions/ADR-007-deterministic-identity-and-gated-automation.md)
by:
1. Removing all 46 superseded historical draft files from `.blueprint/` so zero
   conflicting syntax or error codes remain in the repository.
2. Reserving all V2 and V3 keywords in the V1 lexer (`CORE_V1.md`) so V1 code
   never collides with V2 or V3 language upgrades.
3. Resolving the `T020` cycle diagnostic code (`WOLIO-RESOLVE-001`) and `T024`
   M02 contract clause staging rule (declarative `effects`/`dependencies`/`errors`/
   `intent` are parsed in M02; only `requires`/`ensures` trigger `WOLIO-FEATURE-001`
   at `wolio run` before `T030`).
4. Making all five V1 anti-workaround analyzer codes (`WOLIO-ANALYZER-001`..`005`)
   concrete, enforceable checks (including constant-folding literal checks and
   test-literal mirroring warnings).
5. Making every task file (`T001`–`T074`) self-contained so an agent executing a
   task does not need to read the entire repository.

## Freeze and amendment

Freeze fixes product behavior, feature boundaries, acceptance cases, and schema
versions. An amendment after owner freeze requires an ADR containing old/new
rules, rationale, compatibility, affected fixtures/tasks, and owner decision.
