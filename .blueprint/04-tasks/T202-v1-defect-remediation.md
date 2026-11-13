`.blueprint/04-tasks/T202-v1-defect-remediation.md`
# T202: Fix all V1 compiler, protocol, runtime and native codegen defects

| Field | Value |
|---|---|
| Milestone | `M08` |
| Priority | `P0` |
| Depends on | T201 |
| Blocks | T203 |
| Acceptance families | V2-REGRESSION, all V1 families (`FRONT`..`TOOL`) |
| Bounded areas | `crates/wolio-*`, `tests/conformance/regressions/` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T202.md` |

## Objective
Resolve every V1 compiler, semantic, analyzer, agent protocol, runtime
interpreter, LLVM codegen, package manager, formatter, and LSP defect cataloged
in `T201` so that 100% of `tests/conformance/regressions/BUG-*` and all V1
conformance suites pass in both interpreter and native builds.

## Authority
- Baseline: [BASELINE-V2.md](../BASELINE-V2.md), [BASELINE-V1.md](../BASELINE-V1.md)
- Policy: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md), [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-REGRESSION` and all V1 families (`FRONT`..`TOOL`), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Target crates:** All existing V1 crates (`wolio-diagnostics`, `wolio-ast`, `wolio-lexer`, `wolio-parser`, `wolio-resolver`, `wolio-semantic`, `wolio-types`, `wolio-contracts`, `wolio-effects`, `wolio-analyzer`, `wolio-verify`, `wolio-runtime`, `wolio-ir`, `wolio-codegen`, `wolio-agent`, `wolio-package`, `wolio-format`, `wolio-lsp`, `wolio-cli`).
- **Differential parity rule:** Every `BUG-nnnn` fixture that reaches runtime execution must produce identical stdout, exit code, arithmetic fault (`WOLIO-RUNTIME-001`), and contract violation code/clause (`WOLIO-CONTRACT-002`..`004`) in `wolio run` (interpreter) and `wolio build` (native LLVM binary).
- **No contract/analyzer weakening:** Fixes must never relax `WOLIO-ANALYZER-001`..`005`, `APPROVAL_MATRIX_V1.md` tier computation, or `VerifiedWolioIr` structural checks.

## Scope
In scope:
- Root-cause fixes across `crates/wolio-*` for all `BUG-nnnn` records created in `T201`.
- Updating `.execution/reports/BUG-nnnn.md` with root-cause analysis, changed files, and verification status (`RESOLVED`).
- Full regression and differential parity verification across all V1 conformance families (`FRONT`, `SEM`, `ID`, `RUN`, `CONTRACT`, `EFFECT`, `EVID`, `QUERY`, `TRIAL`, `AGENT`, `ANALYZER`, `NATIVE`, `PKG`, `TOOL`) and `V2-REGRESSION`.

Out of scope (owner):
- `[language] version = "2"` flag enablement (T203).
- Any V2 syntax, types, or protocol v2 operations (T204–T216).

## Normative requirements
- Every `BUG-nnnn` fixture from `T201` must transition from failing (or edge-case reproduction) to passing in CI without modifying or deleting any V1 conformance test (DELIVERY_POLICY, BASELINE-V2).
- Zero open P0 or P1 defects from V1 may remain unresolved at the completion of `T202` (BASELINE-V2, DELIVERY_PLAN).
- Interpreter (`wolio-runtime`) and native codegen (`wolio-codegen`) must maintain 100% behavioral parity across all `BUG-*` and V1 fixtures (DELIVERY_POLICY, ACCEPTANCE_MATRIX).

## Acceptance criteria
Positive and boundary:
- [ ] 100% of `tests/conformance/regressions/BUG-*` fixtures pass in both interpreter and native LLVM execution
- [ ] 100% of V1 conformance families (`FRONT`, `SEM`, `ID`, `RUN`, `CONTRACT`, `EFFECT`, `EVID`, `QUERY`, `TRIAL`, `AGENT`, `ANALYZER`, `NATIVE`, `PKG`, `TOOL`) pass without regression
- [ ] Every `.execution/reports/BUG-nnnn.md` record documents the exact fix and verification result

Rejection and failure:
- [ ] Zero P0 or P1 V1 defects remain open
- [ ] No existing V1 contract, analyzer rule, or IR verification invariant is weakened or bypassed

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No unrelated changes
- [ ] Report written in `.execution/reports/T202.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T202.md` with per-`BUG-nnnn` resolution summary and full workspace test transcript.

## Dependencies and open items
- Depends on: T201; blocks: T203.
- Open: none.

