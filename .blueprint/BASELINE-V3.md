`.blueprint/BASELINE-V3.md`
# Wolio V3 — Industrial Standard Library, Capability I/O, Secrets, Transactions, and Contract Fuzzing

**Status:** planned industrial baseline after V2, 2026-11-10. Design revision: `1.4` (`[language] version = "3"`).  
**Entry prerequisite:** Complete V2 (`M08`–`M11`) acceptance + V2 Bug-Fix & Hardening Gate (`M12`).

## 1. Purpose of V3

After V1 establishes the contract-first core and V2 adds total bounded loops,
collections, `Decimal`, pure predicates, and module architecture enforcement, V3
transforms Wolio into a **self-sufficient industrial backend language** while
closing the remaining avenues for AI technical debt:

1. **V2 Bug-Fix, Performance & Hardening Gate (`M12`):** Fix all defects and
   performance bottlenecks found in V2 loops, collections, formatter, LSP, and
   LLVM lowering before introducing V3 features.
2. **Industrial Standard Library (`std::*`):** Provide battery-included,
   contract-verified modules (`std::string`, `std::collections`, `std::math`,
   `std::json`, `std::time`, `std::crypto`, `std::fs`, `std::net`) so AI never
   invents buggy ad-hoc parsers, encoders, or collection helpers.
3. **Opaque `Secret<T>` & Input Validation Provenance:** Make accidental secret
   logging/serialization a compile-time error (`WOLIO-SECURITY-001`) and enforce
   smart-constructor validation provenance on untrusted external data.
4. **Capability-Gated External I/O & Mandatory Deterministic Test Mocks:** Require
   every I/O effect (`Clock::Read`, `Fs::*`, `Net::*`, `Db::*`) to be authorized in
   `wolio.toml` `[capabilities]` and **forbid live external I/O inside `wolio test` /
   `wolio verify`**, forcing deterministic `with_mock` handlers (`WOLIO-EFFECT-002`).
5. **Linear Resource Handles & Atomic `transaction` Frames:** Prevent aliasing of
   mutable external resources (`resource DbHandle`), enforce `modifies [db]` frames,
   support `old(expr)` pre-state snapshots in `ensures`, and verify rollback
   postconditions on error paths.
6. **Contract-Driven Property Fuzzing (`property` & `wolio verify --fuzz`):**
   Automatically generate hundreds of deterministic boundary and PRNG inputs
   satisfying `requires` to test `ensures`, making it impossible for an AI agent to
   pass verification by hardcoding responses to 2–3 unit test cases.

## 2. Non-collision guarantees with V1 and V2

- All V3 keywords (`secret`, `Secret`, `resource`, `transaction`, `modifies`, `old`,
  `property`, `std`) are reserved from day one in `CORE_V1.md`.
- `std::*` occupies a reserved top-level namespace that user modules in V1 and V2
  cannot declare (`std` is a reserved keyword).
- V3 diagnostics (`WOLIO-SECURITY-001`, `WOLIO-EFFECT-002`, `WOLIO-RESOURCE-001`,
  `WOLIO-FUZZ-001`) are strictly additive.

## 3. V3 delivery profiles (`M12`–`M15`)

| Milestone | Work units | Scope | Exit evidence |
|---|---|---|---|
| `M12` — V2 Bug-Fix, Memory & Performance Gate | `T301`–`T303` | Resolve all V2 bug reports (`BUG-*`); optimize reference-counted `List`/`Map` copy-on-write storage in runtime/LLVM; harden formatter/LSP on large workspaces | Zero open V2 P0/P1 bugs; memory-leak and CoW benchmark gates pass |
| `M13` — Standard Library (`std::*`) & Opaque `Secret<T>` | `T304`–`T307` | Implement pure `std::string`, `std::collections`, `std::math`, `std::json` and `Secret<T>` redaction/sink enforcement (`WOLIO-SECURITY-001`) + `std::crypto` | Conformance suites for all `std::*` pure modules and secret-leak rejection |
| `M14` — Capability I/O, Deterministic Test Mocks & Transactions | `T308`–`T311` | Implement `std::time`, `std::fs`, `std::net`, `resource` non-aliasing (`WOLIO-RESOURCE-001`), `modifies`/`old(...)`, `transaction` rollback verification, and mandatory `with_mock` in tests (`WOLIO-EFFECT-002`) | Fault-injected transaction rollback fixtures; unmocked-test-I/O rejection fixtures |
| `M15` — Contract Property Fuzzing, Agent Protocol v3 & V3 Release | `T312`–`T315` | Implement `property` declarations and `wolio verify --fuzz` shrinking counterexamples; update Agent Protocol v3, formatter, LSP, and native backend; run full W01–W15 industrial benchmark | AI hardcoded-branch defeat fixtures (`WOLIO-FUZZ-001`); V3 Linux x86_64 release and benchmark report |

## 4. Normative V3 documents

- Language grammar and semantics: [CORE_V3.md](02-specifications/language/CORE_V3.md)
- Implementation architecture and crate extensions: [IMPLEMENTATION_SPECIFICATION_V3.md](03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md)
- Standalone task contracts (`T301`–`T315`): [task-index.md](04-tasks/task-index.md) and [V3-TASKS.md](04-tasks/V3-TASKS.md)

