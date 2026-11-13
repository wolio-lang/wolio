`.blueprint/04-tasks/T053-native-build-parity.md`
# T053: Expose `wolio build` and pass interpreter/native release parity

| Field | Value |
|---|---|
| Milestone | `M05` |
| Priority | `P0` |
| Depends on | T052 |
| Blocks | T070 |
| Acceptance families | NATIVE (all); completes M05 |
| Bounded areas | CLI (`crates/wolio-cli`), `tests/native` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T053.md` |

## Objective
Expose native compilation through `wolio build` and prove differential parity
between the interpreter and release-optimized native binaries across the full
conformance corpus, completing M05.

## Authority
- Normative: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md), [CORE_V1.md](../02-specifications/language/CORE_V1.md), [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `NATIVE`, M05 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- `wolio build` CLI command in human and `--json` modes (checking and verifying
  IR before native emission).
- Release-optimized native codegen configuration preserving all arithmetic and
  contract guards.
- Automated differential parity harness (`tests/native`) running all M01–M03
  conformance and trial fixtures through both the interpreter and release
  native builds.

Out of scope (owner):
- Release archive packaging and clean-machine installation (T074).

## Normative requirements
- `run` and `test` use the interpreter by default; `build` explicitly invokes
  native codegen after running full static checks and IR verification, and
  requires a public manifest `[entrypoint]` (`main() -> Unit` or
  `main() -> Int`) (INTERFACE_CONTRACTS, DELIVERY_POLICY).
- Failed builds never produce an executable or reuse stale success evidence;
  `POST_REVIEW` changes that are still `pending` or `rejected` cannot produce a
  release artifact (DELIVERY_POLICY, APPROVAL_MATRIX_V1).
- LLVM optimization passes must preserve all arithmetic faults and contract
  guards in release builds; interpreter and native execution must agree on
  program output, return values, exit codes, fault categories and violated
  contract clause identities across the entire conformance corpus (CORE_V1,
  CONTRACT_DESIGN, DELIVERY_POLICY, DELIVERY_PLAN).

## Acceptance criteria
Positive and boundary:
- [ ] `wolio build` compiles valid projects into runnable Linux x86_64 binaries and emits valid `--json` envelopes
- [ ] Release-optimized native binaries and the interpreter produce identical stdout, result values, exit codes, fault codes and violated clause IDs across all conformance fixtures and M03 trial examples

Rejection and failure:
- [ ] Projects failing static checks, missing a public entrypoint or failing IR verification fail `wolio build` before linking
- [ ] Release optimization never removes or reorders observable `WOLIO-RUNTIME-001` or `WOLIO-CONTRACT-002`/`003`/`004` guards

Completion:
- [ ] All M05 DELIVERY_PLAN and ACCEPTANCE_MATRIX `NATIVE` gates pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p wolio-cli -- build --project <native-fixture-project> --json
```
Run the full conformance corpus under both the interpreter and release-optimized
native backend and compare normalized outputs, faults and violated clause
identities.
Evidence: `.execution/reports/T053.md` with differential parity matrix and binary reproducibility notes.

## Dependencies and open items
- Depends on: T052; blocks: T070.
- Execution prerequisites: pinned LLVM and Linux x86_64 linker toolchain.
- Open: none.

