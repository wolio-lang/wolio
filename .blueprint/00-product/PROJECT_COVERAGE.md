`.blueprint/00-product/PROJECT_COVERAGE.md`
# Whole-project coverage across V1, V2, and V3

Every capability of the Wolio language and AI construction system is assigned to
exactly one non-colliding release baseline (`V1`, `V2`, or `V3`).

| Capability Area | Version & Milestone | Normative Authority | Acceptance Family / Gate |
|---|---|---|---|
| UTF-8 source, V1 grammar, reserved V1/V2/V3 keywords, AST, JSON diagnostics, `init`/`check` | **V1** `M01` (`T001`–`T004`) | [CORE_V1.md](../02-specifications/language/CORE_V1.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md) | `FRONT` |
| Modules, resolution, public visibility, cycle rejection, nominal/record/enum types | **V1** `M02` (`T020`–`T021`) | [CORE_V1.md](../02-specifications/language/CORE_V1.md) | `SEM` |
| `Result`/`Option`, exhaustive `match`, `Result`-handling obligation, checked 64-bit `Int`, interpreter `run` | **V1** `M02` (`T022`, `T024`) | [CORE_V1.md](../02-specifications/language/CORE_V1.md) | `SEM`, `RUN` |
| Deterministic `sym_` IDs, reproducible semantic graph, snapshot & provenance model | **V1** `M02` (`T023`) | [SEMANTIC_MODEL.md](../02-specifications/SEMANTIC_MODEL.md) | `ID` |
| Runtime-guarded `requires`/`ensures`, static `errors`/`effects`/`dependencies` sets | **V1** `M03` (`T030`–`T031`) | [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md) | `CONTRACT`, `EFFECT` |
| `test`/`verify` runner, snapshot-bound evidence, `inspect`/`impact`, compatibility classifier, 3 trial examples & pilot | **V1** `M03` (`T032`–`T034`) | [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md), [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md) | `EVID`, `QUERY`, `TRIAL` |
| Local JSONL agent protocol, typed AST plans/diffs, scope & risk-tier approvals (`AUTO`/`POST_REVIEW`/`HUMAN_REQUIRED`), CAS/journal recovery | **V1** `M04` (`T040`–`T043`) | [LOCAL_PROTOCOL_V1.md](../02-specifications/protocol/LOCAL_PROTOCOL_V1.md), [MUTATION_SCHEMA_V1.md](../02-specifications/protocol/MUTATION_SCHEMA_V1.md), [APPROVAL_MATRIX_V1.md](../02-specifications/protocol/APPROVAL_MATRIX_V1.md) | `AGENT` |
| Anti-workaround analyzer (`WOLIO-ANALYZER-001`..`005`), bounded 3-attempt repair, `finalize`, owner review disposition | **V1** `M04` (`T044`) | [DIAGNOSTIC_SPEC.md](../02-specifications/verification/DIAGNOSTIC_SPEC.md) | `AGENT`, `ANALYZER` |
| Typed SSA-like IR, `VerifiedWolioIr` verifier, pinned LLVM lowering, C/runtime ABI, `build` parity | **V1** `M05` (`T050`–`T053`) | [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) | `NATIVE` |
| 24-task × 10-seed paired comparative benchmark across Wolio, Go, Rust, TypeScript | **V1** `M06` (`T060`–`T063`) | [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md) | `BENCH` |
| Git package resolution, `wolio.lock`, SHA-256 content-addressed cache, `--locked --offline`, `fmt`, `lsp`, Linux x86_64 archive | **V1** `M07` (`T070`–`T074`) | [PACKAGE_MANAGER_SPEC.md](../02-specifications/package-manager/PACKAGE_MANAGER_SPEC.md), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) | `PKG`, `TOOL` |
| **V1 Bug-Fix & Regression Gate** (fix all V1 compiler/protocol/benchmark defects with `BUG-*` conformance fixtures) | **V2** `M08` (`T201`–`T203`) | [BASELINE-V2.md](../BASELINE-V2.md), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) | `V2-REGRESSION` |
| Total bounded `for..in..with..bound..invariant` loops, immutable `List<T>` & `Map<K,V>`, exact `Decimal`, `UInt`, `Bytes`, deterministic `format(...)` | **V2** `M09` (`T204`–`T208`) | [CORE_V2.md](../02-specifications/language/CORE_V2.md) | `V2-LOOP`, `V2-DATA` |
| Reusable pure `predicate` declarations, `[architecture]` module allow/deny rules (`WOLIO-ARCH-001`), multi-module Agent Protocol v2 (`create_module`, `add_import`) | **V2** `M10` (`T209`–`T212`) | [CORE_V2.md](../02-specifications/language/CORE_V2.md), [BASELINE-V2.md](../BASELINE-V2.md) | `V2-PRED`, `V2-ARCH`, `V2-AGENT` |
| V2 Native LLVM lowering, formatter/LSP support for loops/collections/predicates, 24-task algorithmic benchmark | **V2** `M11` (`T213`–`T216`) | [BASELINE-V2.md](../BASELINE-V2.md) | `V2-NATIVE`, `V2-BENCH` |
| **V2 Bug-Fix, Memory & Performance Gate** (CoW `List`/`Map` optimization, large-workspace LSP/formatter hardening) | **V3** `M12` (`T301`–`T303`) | [BASELINE-V3.md](../BASELINE-V3.md) | `V3-REGRESSION` |
| Industrial standard library (`std::string`, `std::collections`, `std::math`, `std::json`, `std::crypto`) & opaque `Secret<T>` (`WOLIO-SECURITY-001`) | **V3** `M13` (`T304`–`T307`) | [CORE_V3.md](../02-specifications/language/CORE_V3.md) | `V3-STD`, `V3-SECRET` |
| Capability-gated external I/O (`std::time`, `std::fs`, `std::net`), mandatory `with_mock` in tests (`WOLIO-EFFECT-002`), linear `resource` handles & `transaction` rollback (`WOLIO-RESOURCE-001`) | **V3** `M14` (`T308`–`T311`) | [CORE_V3.md](../02-specifications/language/CORE_V3.md) | `V3-IO`, `V3-TX` |
| Contract-driven property fuzzing (`property`, `wolio verify --fuzz`, `WOLIO-FUZZ-001`), Agent Protocol v3, full W01–W15 industrial evaluation & release | **V3** `M15` (`T312`–`T315`) | [CORE_V3.md](../02-specifications/language/CORE_V3.md), [BASELINE-V3.md](../BASELINE-V3.md) | `V3-FUZZ`, `V3-RELEASE` |
| Unbounded `while (true)` loops, IEEE-754 binary floats in business logic, implicit null/truthiness, inheritance, macros, raw text/regex agent patches | **Permanently Excluded** | [BASELINE-V1.md](../BASELINE-V1.md), [CORE_V2.md](../02-specifications/language/CORE_V2.md) | Rejected by design |

## Completion definitions

- **M03 Trial:** Frontend + executable core + runtime-guarded contracts + runnable
  examples + 6-task pilot evidence; no dependency on LLVM or agent automation.
- **V1 Complete:** All `M01`–`M07` (`T001`–`T074`) accepted.
- **V2 Complete:** All `M08`–`M11` (`T201`–`T216`) accepted after V1 bug-fix gate.
- **V3 Complete:** All `M12`–`M15` (`T301`–`T315`) accepted after V2 bug-fix gate.
