`.blueprint/04-tasks/EXECUTION_CONTRACTS.md`
# V1–V3 task execution rules

Shared rules for every registered Wolio task across V1 (`T001`–`T074`), V2
(`T201`–`T216`), and V3 (`T301`–`T315`). Every V1 (`T001`–`T074`), V2
(`T201`–`T216`), and V3 (`T301`–`T315`) task's complete contract lives in its
own self-contained file registered in [task-index.md](task-index.md) (with V2
and V3 also summarized in [V2-TASKS.md](V2-TASKS.md) and
[V3-TASKS.md](V3-TASKS.md)). Actual readiness and live states belong only to
`.execution/task-status.md`.

## 1. Self-contained task execution principle

Every task file is engineered so an AI coding agent or human engineer can execute
that single task **without reading the entire repository**:
- Each task file embeds its **Self-contained implementation context** (exact
  crate path, allowed Cargo dependencies, required AST/data structures, exact
  diagnostic codes, and CLI/JSON behavior).
- You only need to read:
  1. `.execution/current-state.md` and `.execution/task-status.md` (to confirm readiness),
  2. The assigned task file (`04-tasks/Tnnn-*.md`), and
  3. Only if a deeper grammar/protocol edge case is needed, the specific normative
     file linked under `## Authority`.

## 2. Single source per fact

| Fact | Sole source |
|---|---|
| Language, protocol, package, and evidence behavior | Normative documents listed in [../DOCUMENT_AUTHORITY.md](../DOCUMENT_AUTHORITY.md) |
| Acceptance family definitions | [../05-quality/ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) |
| Milestone order and milestone gates | [../06-project-management/DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md) |
| One task's objective, scope, dependencies, self-contained rules, and verification | That task's contract (`Tnnn-*.md`, `V2-TASKS.md`, or `V3-TASKS.md`) |
| Task ID allocation and file location | [task-index.md](task-index.md) |
| Live status, blockers, decisions, and evidence | Repository `.execution/` |

On any difference between a task summary and a cited normative specification, stop
the affected task, record the exact conflict under `.execution/blockers/`, and
fix it in one documentation amendment.

## 3. Task registry, version blocks, and bug-fix control

- **Numbering blocks:**
  - **V1 (`M01`–`M07`):** `M01` `T001`–`T019`, `M02` `T020`–`T029`, `M03` `T030`–`T039`, `M04` `T040`–`T049`, `M05` `T050`–`T059`, `M06` `T060`–`T069`, `M07` `T070`–`T079`.
  - **V2 (`M08`–`M11`):** `M08` `T201`–`T203` (V1 Bug-Fix Gate), `M09` `T204`–`T208` (Bounded Loops & Collections), `M10` `T209`–`T212` (Predicates, Architecture & Agent v2), `M11` `T213`–`T216` (V2 Native & Benchmark).
  - **V3 (`M12`–`M15`):** `M12` `T301`–`T303` (V2 Bug-Fix & Perf Gate), `M13` `T304`–`T307` (`std::*` & `Secret<T>`), `M14` `T308`–`T311` (Capability I/O & Transactions), `M15` `T312`–`T315` (Property Fuzzing & V3 Release).
- **Bug-fix records (`BUG-nnnn`):** Defects discovered during or after a task use
  `.execution/reports/BUG-nnnn.md` and must add a permanent failing-then-passing
  regression fixture under `tests/conformance/regressions/BUG-nnnn/`.
- IDs are never reused, renumbered, or repurposed. Every behavior has exactly one
  implementing task. When a later task needs an earlier task's analysis, it
  consumes that analysis instead of reimplementing it.

## 4. Common gates

Every Rust work unit runs:
- `cargo fmt --all -- --check`
- `cargo clippy --workspace --all-targets -- -D warnings`
- `cargo test --workspace`
(`T001` additionally runs `cargo check --workspace`).
Store compiler conformance fixtures under `tests/conformance/<family>/`, executed
by host-side tests through the real frontend/runtime. Keep tests independent of
network except explicitly marked Git integration tests with controlled local
repositories; never require production credentials in CI.

## 5. Readiness and report contract

`READY` means the task contract exists, every `Depends on` task is `DONE`, and
the owner baseline for that version is accepted in `.execution/baseline-acceptance.md`.
Missing external prerequisites (LLVM toolchain for `M05`, model API access/budget
for `T034`/`M06`, owner license decision for `T074`) make the task `BLOCKED` with
a record under `.execution/blockers/`; they never permit simulated evidence.

Record implementation reports in `.execution/reports/Tnnn.md` and preparation
reports in `.execution/reports/PREP-nnnn.md`: task ID, baseline identity, changed
paths, implemented behavior, commands/exit results, fixture families, limitations,
and follow-up.
