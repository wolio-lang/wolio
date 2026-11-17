`.blueprint/04-tasks/T216-v2-release-distribution.md`
# T216: Package and verify Wolio V2 Linux x86_64 release distribution

| Field | Value |
|---|---|
| Milestone | `M11` |
| Priority | `P0` |
| Depends on | T215 |
| Blocks | T301 (V3 M12 entry) |
| Acceptance families | V2-NATIVE, TOOL; completes M11 and full V2 |
| Bounded areas | release packaging, user/install documentation, CI workflows |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T216.md` |

## Objective
Package the reproducible Wolio V2 Linux x86_64 `.tar.gz` distribution archive
with SHA-256 checksums, pinned toolchain metadata, license, and V2 release
notes, and verify clean-machine installation for both `[language] version = "1"`
and `[language] version = "2"` workflows, completing M11 and V2.

## Authority
- Normative: [BASELINE-V2.md](../BASELINE-V2.md), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md), [RELEASE_CRITERIA.md](../05-quality/RELEASE_CRITERIA.md), [APPROVAL_MATRIX_V1.md](../02-specifications/protocol/APPROVAL_MATRIX_V1.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-NATIVE` and `TOOL`, M11 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Release bundle contents:**
  - `wolio` binary (supporting `[language] version = "1"` and `"2"`, `protocol_version: 1` and `2`, native `build`, `fmt`, `lsp`, and package manager commands).
  - `SHA256SUMS` manifest covering the archive and binary.
  - `TOOLCHAIN.json` recording exact pinned Rust version, `Cargo.lock` SHA-256, LLVM version/digest, and target triple (`x86_64-unknown-linux-gnu`).
  - `LICENSE` and `RELEASE_NOTES_V2.md` (summarizing `M08` bug fixes, `M09` bounded loops/collections/`Decimal`, `M10` predicates/`[architecture]`/Protocol v2, and `M11` native parity & benchmark findings).
- **Clean-machine verification:**
  - Verify that a fresh Linux x86_64 environment without a Rust build tree can unpack the archive, verify `SHA256SUMS`, run a V1 project (`version = "1"`) with unchanged hashes, and run/test/verify/fmt/build a V2 project (`version = "2"`) using bounded `for` loops, `List`/`Map`, `Decimal`, `predicate`, and `[architecture]` rules.

## Scope
In scope:
- Building and verifying the V2 Linux x86_64 `.tar.gz` release archive and `SHA256SUMS`.
- Verifying release eligibility gates (zero `pending` or `rejected` `POST_REVIEW` records, 100% of V1 and V2 conformance and `BUG-*` regression suites passing).
- Clean-environment installation test running both V1 (`version = "1"`) and V2 (`version = "2"`) end-to-end sample workflows.

Out of scope (owner):
- V3 (`M12`–`M15`, `T301`–`T319`) features (`std::*`, `Secret<T>`, capability I/O, FFI, transactions, property fuzzing).

## Normative requirements
- Release artifacts require a Linux x86_64 `.tar.gz`, SHA-256 checksums, exact pinned toolchain/lock metadata, license, and release notes (DELIVERY_POLICY, RELEASE_CRITERIA).
- Any `pending` or `rejected` `POST_REVIEW` record in `.wolio-log/` or any failing V1/V2 conformance or `BUG-*` regression test blocks release publication (APPROVAL_MATRIX_V1, DELIVERY_POLICY).
- Clean-machine installation must verify both upward V1 compatibility (`[language] version = "1"`) and full V2 capability (`[language] version = "2"`) across `init`, `check`, `run`, `test`, `verify`, `fmt`, and `build` (BASELINE-V2, DELIVERY_POLICY).

## Acceptance criteria
Positive and boundary:
- [ ] V2 Linux x86_64 `.tar.gz` archive, `SHA256SUMS`, `TOOLCHAIN.json`, `LICENSE`, and `RELEASE_NOTES_V2.md` are generated reproducibly
- [ ] Clean-machine install passes checksum verification and executes both a V1 sample workflow (`version = "1"`) and a V2 sample workflow (`version = "2"`) through `check`, `run`, `test`, `verify`, `fmt`, and native `build`

Rejection and failure:
- [ ] Corrupted archive bytes or mismatched SHA-256 checksums are rejected by verification scripts
- [ ] Any `pending` or `rejected` `POST_REVIEW` record blocks release packaging

Completion:
- [ ] All M08–M11 gates in `DELIVERY_PLAN` and `ACCEPTANCE_MATRIX` (`V2-REGRESSION`, `V2-LOOP`, `V2-DATA`, `V2-PRED`, `V2-ARCH`, `V2-AGENT`, `V2-NATIVE`, `V2-BENCH`) pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T216.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Install the V2 `.tar.gz` archive in a clean Linux x86_64 environment, verify
`SHA256SUMS`, and execute the V1 and V2 end-to-end workflows.
Evidence: `.execution/reports/T216.md` with archive SHA-256 digests and clean-install verification transcript.

## Dependencies and open items
- Depends on: T215; blocks: T301 (V3 M12 entry).
- Open: none.

