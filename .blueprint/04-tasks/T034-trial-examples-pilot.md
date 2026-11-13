`.blueprint/04-tasks/T034-trial-examples-pilot.md`
# T034: Deliver M03 trial examples, user documentation and pilot benchmark

| Field | Value |
|---|---|
| Milestone | `M03` |
| Priority | `P0` |
| Depends on | T033 |
| Blocks | T040, T050 |
| Acceptance families | TRIAL; completes M03 |
| Bounded areas | examples, user documentation, `benchmarks/pilot` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T034.md` |

## Objective
Ship the three runnable M03 trial examples, three sequential change cycles,
fresh-user trial documentation and the six-task pilot benchmark with raw
results, completing the M03 trial stop point.

## Authority
- Normative: [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md) (Trial stop point and Benchmark protocol), [RELEASE_CRITERIA.md](../05-quality/RELEASE_CRITERIA.md) (M03 trial), [CORE_V1.md](../02-specifications/language/CORE_V1.md), [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `TRIAL`, M03 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Taxonomy & Implementation: [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md), [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Three runnable examples: bounded discount calculation, nominal-ID lookup
  returning typed errors, and a multi-operation order total.
- Fixture demonstrating a wrong implementation that passes type checking but
  fails `ensures`, plus three sequential change cycles with `impact`/`verify`.
- Fresh-user installation and usage documentation covering `init`, `check`,
  `run`, `test`, `inspect`, `impact` and `verify` without LLVM or agent
  orchestration, stating actual V1 limitations.
- Six-task, three-run-per-language descriptive pilot comparing Wolio and
  TypeScript (with optional Go/Rust arms) and publishing raw run records.

Out of scope (owner):
- Local agent mutation protocol (T040–T044); native LLVM backend (T050–T053);
  24-task expanded evaluation (T060–T063).

## Normative requirements
- M03 ships three runnable examples (bounded discount, nominal-ID lookup with
  typed errors, multi-operation order total), a type-valid implementation that
  fails `ensures`, and three sequential contract/implementation change cycles
  (DELIVERY_PLAN).
- A fresh user can `init`, `check`, `run`, `test`, `inspect`, `impact` and
  `verify` from pinned Cargo source without LLVM or agent tooling; documentation
  states actual limitations and makes no claim of universal proof
  (DELIVERY_PLAN, RELEASE_CRITERIA).
- Pilot benchmark runs 6 tasks (implement, edge-case fix, wrong nominal ID,
  missing error branch, contract change, multi-operation regression) × 3 runs
  per language (Wolio and strict TypeScript; Go/Rust optional) under a frozen
  prompt/budget/model configuration (3 repair attempts, 10 minutes/task),
  retaining all failures, timeouts, hashes and raw outputs (DELIVERY_PLAN).
- Pilot results are descriptive only; negative findings are published without
  selective omission. Missing model API access/budget makes the pilot BLOCKED,
  never simulated (DELIVERY_PLAN, EXECUTION_CONTRACTS).

## Acceptance criteria
Positive and boundary:
- [ ] All three trial examples pass `check`, `run`, `test`, `inspect`, `impact` and `verify` from a clean directory
- [ ] Three sequential change cycles produce expected `impact` diffs and updated `verify` evidence
- [ ] Six-task × three-run pilot harness records prompts, tool/model versions, hashes and raw outcomes

Rejection and failure:
- [ ] Wrong implementation fixture passes `check` (types) and fails `ensures` (`WOLIO-CONTRACT-003`) at runtime/test
- [ ] Unsuccessful pilot runs, timeouts and limitations are retained and documented without omission

Completion:
- [ ] All M03 DELIVERY_PLAN and RELEASE_CRITERIA trial gates pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Execute every documented trial command (`init`, `check`, `run`, `test`,
`inspect`, `impact`, `verify`) from a fresh directory and run the six-task,
three-run-per-language pilot defined in `DELIVERY_PLAN.md`.
Evidence: `.execution/reports/T034.md` with trial command transcripts and raw pilot records.

## Dependencies and open items
- Depends on: T033; blocks: T040, T050.
- Execution prerequisites: pinned Rust toolchain; model API access and budget
  for pilot benchmark runs (record a blocker under `.execution/blockers/` if
  unavailable).
- Open: none.

