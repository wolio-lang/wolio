`.blueprint/README.md`

# Wolio Blueprint (Revision 1.4)

This directory is the authoritative engineering specification for **Wolio**, an
industrial contract-first programming language and AI software construction
system designed to prevent AI reward-hacking and workarounds.

## Document authority and zero-clutter layout

[DOCUMENT_AUTHORITY.md](DOCUMENT_AUTHORITY.md) is the controlling register. All
46 superseded historical draft files have been removed in Revision 1.4 so that
every file in `.blueprint/` is current and non-contradictory.

1. **Baselines:**
   - [BASELINE-V1.md](BASELINE-V1.md): V1 Immutable Contract Core, Local Agent Protocol, Verified LLVM Backend, Benchmark & Toolchain (`M01`–`M07`, `T001`–`T074`).
   - [BASELINE-V2.md](BASELINE-V2.md): V2 V1-Bugfix Gate, Total Bounded Loops, `List`/`Map`, Exact `Decimal`, Reusable Pure `predicate`s & `[architecture]` Policy (`M08`–`M11`, `T201`–`T216`).
   - [BASELINE-V3.md](BASELINE-V3.md): V3 V2-Bugfix Gate, Industrial Standard Library `std::*`, `Secret<T>`, Capability-Gated I/O Mocks, `transaction` Frames & Contract Property Fuzzing (`M12`–`M15`, `T301`–`T315`).
2. **Layers:**
   - `00-product/`: Whole-project coverage across V1, V2, and V3.
   - `01-architecture/`: Architecture Decision Records (`ADR-005`..`ADR-008`).
   - `02-specifications/`: Normative language (`CORE_V1`, `CORE_V2`, `CORE_V3`), contract, semantic, protocol, package, and diagnostic specifications.
   - `03-implementation/`: Self-contained crate boundaries (`IMPLEMENTATION_SPECIFICATION_V1.md`, `IMPLEMENTATION_SPECIFICATION_V2.md`, `IMPLEMENTATION_SPECIFICATION_V3.md`) and delivery/bug-fix policies (`DELIVERY_POLICY.md`).
   - `04-tasks/`: Self-contained task contracts (`T001`–`T074`, `T201`–`T216` in `V2-TASKS.md`, `T301`–`T315` in `V3-TASKS.md`).
   - `05-quality/`: Acceptance matrix, Definition of Done, and Release Criteria.
   - `06-project-management/`: Delivery plan, Post-V1 map, and `AI_BUG_TAXONOMY.md` (`W01`–`W15`).
   - `07-ai-context/`: Agent execution rules and Ubuntu quickstart.

## Fast path for coding agents

1. Read `.execution/current-state.md` and `.execution/task-status.md`.
2. Open the assigned task file in `04-tasks/`. Every task file contains a
   **Self-contained implementation context** section so you do not need to read
   the entire repository.
3. Implement only the assigned task scope, run its verification commands, and
   record evidence in `.execution/reports/Tnnn.md`.
