`.blueprint/04-tasks/T201-v1-defect-triage-regressions.md`
# T201: Triage V1 benchmark and implementation defects and build `BUG-*` regression suite

| Field | Value |
|---|---|
| Milestone | `M08` |
| Priority | `P0` |
| Depends on | T063, T074, and owner acceptance of the V2 M08 gate |
| Blocks | T202 |
| Acceptance families | V2-REGRESSION (triage and regression fixtures) |
| Bounded areas | `tests/conformance/regressions/`, `.execution/reports/BUG-*.md`, `crates/wolio-verify` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T201.md` |

## Objective
Audit every defect, compiler edge case, diagnostic mismatch, and benchmark
escape discovered during V1 (`M01`–`M07`, including `T034` trial and `T062`
10-task Contract & Protocol benchmark) and encode each defect as a deterministic, reproducible
conformance fixture under `tests/conformance/regressions/BUG-nnnn/` before any
V2 feature work begins.

## Authority
- Baseline: [DOCUMENT_AUTHORITY.md](../DOCUMENT_AUTHORITY.md), [BASELINE-V2.md](../BASELINE-V2.md)
- Policy: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) (Section 3: Cross-version Bug-Fix & Regression Gate Policy), [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-REGRESSION`, M08 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Regression fixture directory layout:**
  - `tests/conformance/regressions/BUG-nnnn/wolio.toml`
  - `tests/conformance/regressions/BUG-nnnn/src/main.wol`
  - `tests/conformance/regressions/BUG-nnnn/expected.json` (pinning exact expected command, exit code, diagnostics array, and runtime output/fault)
- **Bug report record layout:**
  - `.execution/reports/BUG-nnnn.md` recording: defect ID, originating V1 task or benchmark run (`T001`–`T074`), root-cause subsystem (`lexer`, `parser`, `semantic` [submodules `resolver`, `types`, `graph`, `contracts`, `effects`, `analyzer`], `agent`, `ir`, `codegen`, `package`, `format`, or `lsp`), reproduction fixture path, and severity (`P0`/`P1`/`P2`).
- **Host runner integration:**
  - `crates/wolio-verify` and workspace conformance test harness discover and run all `tests/conformance/regressions/BUG-*` fixtures in deterministic lexicographical order.

## Scope
In scope:
- Complete triage of all V1 execution reports (`.execution/reports/T001.md`..`T074.md`), blocker logs, and M06 benchmark transcripts.
- Creation of `.execution/reports/BUG-nnnn.md` records and `tests/conformance/regressions/BUG-nnnn/` fixtures reproducing each defect.
- Automated regression test harness executing `tests/conformance/regressions/BUG-*` across both interpreter and native test runners.

Out of scope (owner):
- Fixing the underlying compiler/runtime/codegen defects (T202).
- `[language] version = "2"` manifest gating (T203) or any V2 syntax/types (T204 onward).

## Normative requirements
- `M08` is a mandatory stabilization gate before `M09`: no V2 grammar or type feature may be implemented before all V1 defects are cataloged and reproduced (BASELINE-V2, DELIVERY_POLICY).
- Every bug discovered during V1 milestone execution or benchmark campaigns (`M03`, `M06`) must have a `.execution/reports/BUG-nnnn.md` record and a minimal conformance fixture under `tests/conformance/regressions/BUG-nnnn/` (DELIVERY_POLICY, EXECUTION_CONTRACTS).
- Regression fixtures must be self-contained, hermetic (zero network access), and assert exact diagnostic codes, byte spans, or runtime outputs (EXECUTION_CONTRACTS, ACCEPTANCE_MATRIX).

## Acceptance criteria
Positive and boundary:
- [ ] All V1 reports (`T001`–`T074`), pilot findings (`T034`), and 24-task evaluation logs (`T062`–`T063`) are audited and indexed into `.execution/reports/BUG-nnnn.md`
- [ ] Every cataloged defect has a minimal reproducible fixture under `tests/conformance/regressions/BUG-nnnn/` with deterministic `expected.json` assertions
- [ ] Even if zero open P0/P1 defects remain from V1, at least the boundary edge-case fixtures identified during M04/M05/M06 audit are locked into `tests/conformance/regressions/`

Rejection and failure:
- [ ] Regression harness rejects malformed `BUG-nnnn` directories missing `expected.json` or deterministic exit-code assertions
- [ ] Fixtures do not weaken any existing V1 `FRONT`..`TOOL` acceptance test

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No V2 language semantics introduced; no unrelated changes
- [ ] Report written in `.execution/reports/T201.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace -- --nocapture regressions
```
Evidence: `.execution/reports/T201.md` with the complete `BUG-nnnn` inventory table and fixture paths.

## Dependencies and open items
- Depends on: T063, T074, and owner acceptance of V2 M08 gate; blocks: T202.
- Open: none.

