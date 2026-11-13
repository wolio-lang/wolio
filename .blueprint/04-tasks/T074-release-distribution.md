`.blueprint/04-tasks/T074-release-distribution.md`
# T074: Package Linux x86_64 release artifacts, checksums and clean-install verification

| Field | Value |
|---|---|
| Milestone | `M07` |
| Priority | `P0` |
| Depends on | T073 |
| Blocks | none |
| Acceptance families | TOOL (clean install); completes M07 and full V1 |
| Bounded areas | release packaging, user/install documentation, CI workflows |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T074.md` |

## Objective
Build the reproducible Linux x86_64 `.tar.gz` release distribution with SHA-256
checksums, toolchain/lock metadata, owner-approved license and release notes,
and verify clean-machine installation, completing M07 and V1.

## Authority
- Normative: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) (Manifest and versions), [RELEASE_CRITERIA.md](../05-quality/RELEASE_CRITERIA.md) (Full V1), [APPROVAL_MATRIX_V1.md](../02-specifications/protocol/APPROVAL_MATRIX_V1.md), [BASELINE-V1.md](../BASELINE-V1.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `TOOL`, M07 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [RELEASE_CRITERIA.md](../05-quality/RELEASE_CRITERIA.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Release packaging script and CI workflow producing the Linux x86_64 `.tar.gz`
  binary archive, SHA-256 checksum manifest, pinned Rust/LLVM/`Cargo.lock`
  metadata, release notes and the owner-selected license file.
- Release eligibility gate verifying that all M01–M07 checks pass and no
  `POST_REVIEW` record is `pending` or `rejected`.
- Clean-environment Linux x86_64 installation verification and corrupt-archive/
  checksum-mismatch rejection tests.

Out of scope (owner):
- Non-Linux or non-x86_64 target distributions (requires a separate platform
  support amendment under `PROJECT_COVERAGE.md`).

## Normative requirements
- Release artifacts consist of a Linux x86_64 `.tar.gz` archive, SHA-256
  checksums, exact toolchain/lock metadata, the owner-selected project license
  and release notes; never invent ownership or a license grant (DELIVERY_POLICY,
  RELEASE_CRITERIA, EXECUTION_CONTRACTS).
- No floating tool version is permitted in recorded release evidence; failed
  builds or `pending`/`rejected` `POST_REVIEW` records block release artifact
  publication (DELIVERY_POLICY, APPROVAL_MATRIX_V1).
- Clean-machine binary installation on Linux x86_64 must verify SHA-256
  checksums, reject corrupt archives or mismatched checksums, and execute the
  documented sample workflow without requiring a repository source build
  (ACCEPTANCE_MATRIX, DELIVERY_PLAN, RELEASE_CRITERIA).

## Acceptance criteria
Positive and boundary:
- [ ] Release packaging produces the Linux x86_64 `.tar.gz`, SHA-256 checksum file, pinned toolchain metadata, release notes and owner-selected license
- [ ] Installing the built archive in a clean Linux x86_64 environment passes checksum validation and runs the documented sample workflow (`init`, `check`, `run`, `test`, `verify`, `fmt`, `build`)

Rejection and failure:
- [ ] Corrupt archive bytes or modified SHA-256 checksums are rejected by the installer/verification check
- [ ] Missing owner license decision or any `pending`/`rejected` `POST_REVIEW` record blocks release packaging

Completion:
- [ ] All M07 and full V1 gates in DELIVERY_PLAN, ACCEPTANCE_MATRIX and RELEASE_CRITERIA pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Install the built `.tar.gz` archive in a fresh Linux x86_64 environment,
validate SHA-256 checksums, test corrupt-archive rejection, and run the
documented sample workflow.
Evidence: `.execution/reports/T074.md` with archive SHA-256 digests, pinned toolchain versions, owner license reference and clean-install transcript.

## Dependencies and open items
- Depends on: T073; blocks: none.
- Execution prerequisites: owner license decision recorded under
  `.execution/decisions/` before publication (record a blocker under
  `.execution/blockers/` if absent).
- Open: none.

