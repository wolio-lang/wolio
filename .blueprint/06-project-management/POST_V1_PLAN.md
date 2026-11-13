`.blueprint/06-project-management/POST_V1_PLAN.md`
# Post-V1 product roadmap (V2 and V3 baselines)

The earlier exploratory work packages (`E01`–`E10`) are now structured into two
concrete, non-colliding release blueprints: [BASELINE-V2.md](../BASELINE-V2.md)
(`[language] version = "2"`) and [BASELINE-V3.md](../BASELINE-V3.md)
(`[language] version = "3"`).

## Mapping of historical E01–E10 packages to V2 and V3

| Former ID | Capability | Assigned Version & Milestone | Normative Specification & Tasks |
|---|---|---|---|
| — | V1 Bug-Fix & Regression Hardening Gate | **V2** `M08` (`T201`–`T203`) | [BASELINE-V2.md](../BASELINE-V2.md), [V2-TASKS.md](../04-tasks/V2-TASKS.md) |
| `E10` (core subset) | Total bounded loops (`for..in..with..bound..invariant`), `List<T>`, `Map<K,V>`, `Decimal`, `UInt`, `Bytes`, `format(...)` | **V2** `M09` (`T204`–`T208`) | [CORE_V2.md](../02-specifications/language/CORE_V2.md), [V2-TASKS.md](../04-tasks/V2-TASKS.md) |
| `E06` | Reusable pure `predicate`s, `[architecture]` allow/deny rules (`WOLIO-ARCH-001`), multi-module Agent Protocol v2 | **V2** `M10` (`T209`–`T212`) | [CORE_V2.md](../02-specifications/language/CORE_V2.md), [V2-TASKS.md](../04-tasks/V2-TASKS.md) |
| — | V2 Native LLVM parity, V2 Formatter/LSP & 24-task Algorithmic Benchmark | **V2** `M11` (`T213`–`T216`) | [BASELINE-V2.md](../BASELINE-V2.md), [V2-TASKS.md](../04-tasks/V2-TASKS.md) |
| — | V2 Bug-Fix, Copy-on-Write Memory & Performance Gate | **V3** `M12` (`T301`–`T303`) | [BASELINE-V3.md](../BASELINE-V3.md), [V3-TASKS.md](../04-tasks/V3-TASKS.md) |
| `E05` | Industrial Standard Library (`std::*`), opaque `Secret<T>` (`WOLIO-SECURITY-001`), validation provenance | **V3** `M13` (`T304`–`T307`) | [CORE_V3.md](../02-specifications/language/CORE_V3.md), [V3-TASKS.md](../04-tasks/V3-TASKS.md) |
| `E01`, `E02`, `E08` | Capability-gated external I/O, mandatory `with_mock` test sandbox (`WOLIO-EFFECT-002`), linear `resource` handles & `transaction` rollback (`WOLIO-RESOURCE-001`) | **V3** `M14` (`T308`–`T311`) | [CORE_V3.md](../02-specifications/language/CORE_V3.md), [V3-TASKS.md](../04-tasks/V3-TASKS.md) |
| `E07` | Contract-driven property fuzzing (`property`, `wolio verify --fuzz`, `WOLIO-FUZZ-001`), Agent Protocol v3 & Full Industrial Benchmark | **V3** `M15` (`T312`–`T315`) | [CORE_V3.md](../02-specifications/language/CORE_V3.md), [V3-TASKS.md](../04-tasks/V3-TASKS.md) |
| `E03`, `E04`, `E09` | Shared-memory/async concurrency, distributed idempotency daemons, remote multi-tenant agent server | Deferred Post-V3 research | Requires separate Post-V3 ADR after `M15` |
| `E10` (excluded) | Unbounded `while (true)` loops, IEEE-754 binary floats in business logic, inheritance, macros | Permanently rejected | Violates deterministic AI anti-workaround guarantees |
