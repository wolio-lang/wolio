`.blueprint/04-tasks/T301-v2-defect-triage-regressions.md`
# T301: Triage V2 defects and build `BUG-*` V2 regression suite

| Field | Value |
|---|---|
| Milestone | `M12` |
| Priority | `P0` |
| Depends on | T216, and owner acceptance of the V3 M12 gate |
| Blocks | T302 |
| Acceptance families | V3-REGRESSION |
| Bounded areas | `tests/conformance/regressions/`, `.execution/reports/BUG-*.md`, `crates/wolio-*` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T301.md` |

## Objective
Audit every compiler, bounded-loop verifier, collection, `Decimal`, `predicate`,
`[architecture]`, Agent Protocol v2, formatter, LSP, and native LLVM parity
defect discovered during V2 (`M09`–`M11`, including the `T215` 24-task
algorithmic benchmark), encode each defect as a permanent reproducible
conformance fixture under `tests/conformance/regressions/BUG-nnnn/`, and resolve
every underlying root cause across the interpreter and native LLVM backend
before any V3 feature work begins.

## Authority
- Baseline: [DOCUMENT_AUTHORITY.md](../DOCUMENT_AUTHORITY.md), [BASELINE-V3.md](../BASELINE-V3.md), [BASELINE-V2.md](../BASELINE-V2.md)
- Policy: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) (Section 3: Cross-version Bug-Fix & Regression Gate Policy), [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-REGRESSION`, M12 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Regression fixture directory layout:**
  - `tests/conformance/regressions/BUG-nnnn/wolio.toml` (`[language] version = "1"` or `"2"`)
  - `tests/conformance/regressions/BUG-nnnn/src/main.wol` (plus any multi-module `src/*.wol` files for `[architecture]` / Protocol v2 bugs)
  - `tests/conformance/regressions/BUG-nnnn/expected.json` (pinning exact command, exit code, diagnostics array, and stdout/fault behavior)
- **Bug report record layout:**
  - `.execution/reports/BUG-nnnn.md` recording: defect ID, originating V2 task or benchmark trial (`T204`–`T216`), affected crate(s), root-cause analysis, reproduction fixture path, fix summary, and verification status (`RESOLVED`).
- **Differential parity requirement:**
  - Every `BUG-nnnn` fixture that reaches runtime execution must produce identical stdout, exit code, arithmetic fault (`WOLIO-RUNTIME-001`), and contract/loop violation code (`WOLIO-CONTRACT-002`..`006`) in both `wolio run` (interpreter) and `wolio build` (native LLVM binary).

## Scope
In scope:
- Complete triage of all V2 execution reports (`.execution/reports/T201.md`..`T216.md`), blocker logs, and `T215` V2 benchmark transcripts.
- Authoring `.execution/reports/BUG-nnnn.md` records and `tests/conformance/regressions/BUG-nnnn/` failing-then-passing conformance fixtures.
- Fixing all cataloged V2 defects across `crates/wolio-*` without weakening any V1 or V2 contract, loop guard, `[architecture]` rule, or analyzer check.

Out of scope (owner):
- Copy-on-Write (CoW) reference-counting optimization for `List<T>` and `Map<K, V>` (T302).
- `[language] version = "3"` manifest gating and large-workspace LSP profiling (T303).
- Any V3 `std::*`, `Secret<T>`, capability I/O, `resource`/`transaction`, or `property` fuzzing features (T304–T315).

## Normative requirements
- `M12` is a mandatory stabilization gate before `M13`: no V3 standard library or language construct may be implemented until all open V2 P0/P1 defects are reproduced in `tests/conformance/regressions/BUG-nnnn/` and fixed (BASELINE-V3, DELIVERY_POLICY, EXTENSION_BOUNDARIES).
- Every bug fix must preserve 100% of existing V1 (`FRONT`..`TOOL`) and V2 (`V2-REGRESSION`, `V2-LOOP`, `V2-DATA`, `V2-PRED`, `V2-ARCH`, `V2-AGENT`, `V2-NATIVE`) conformance suites (DELIVERY_POLICY, ACCEPTANCE_MATRIX).
- Interpreter (`wolio-runtime`) and native LLVM (`wolio-codegen`) backends must agree 100% across all `BUG-*` fixtures (DELIVERY_POLICY).

## Acceptance criteria
Positive and boundary:
- [ ] All V2 reports (`T201`–`T216`) and `T215` benchmark failure logs are audited and cataloged in `.execution/reports/BUG-nnnn.md`
- [ ] Every cataloged V2 defect has a deterministic conformance fixture under `tests/conformance/regressions/BUG-nnnn/` that passes after remediation in both interpreter and native LLVM execution
- [ ] All V1 and V2 conformance suites continue to pass without regression

Rejection and failure:
- [ ] Zero open P0 or P1 defects from V1 or V2 remain unresolved
- [ ] Regression runner rejects any `BUG-nnnn` fixture missing `expected.json` or deterministic assertions
- [ ] No V1 or V2 contract guard, loop verifier rule, or anti-workaround analyzer check (`WOLIO-ANALYZER-001`..`005`) is weakened

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T301.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace -- --nocapture regressions
cargo test --workspace
```
Evidence: `.execution/reports/T301.md` with the complete V2 `BUG-nnnn` inventory, root-cause fixes, and differential test results.

## Dependencies and open items
- Depends on: T216, and owner acceptance of the V3 M12 gate; blocks: T302.
- Open: none.

