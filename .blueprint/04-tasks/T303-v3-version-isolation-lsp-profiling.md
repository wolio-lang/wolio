`.blueprint/04-tasks/T303-v3-version-isolation-lsp-profiling.md`
# T303: Enforce `[language] version = "1" | "2" | "3"` isolation and large-workspace LSP profiling

| Field | Value |
|---|---|
| Milestone | `M12` |
| Priority | `P0` |
| Depends on | T302 |
| Blocks | T304 (M13 entry) |
| Acceptance families | V3-REGRESSION; completes M12 |
| Bounded areas | `crates/wolio-diagnostics`, `crates/wolio-ast`, `crates/wolio-parser`, `crates/wolio-semantic`, `crates/wolio-lsp`, `crates/wolio-cli` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T303.md` |

## Objective
Enable `[language] version = "3"` in `wolio.toml` and compiler configuration
while strictly isolating `[language] version = "1"` and `"2"` projects so they
reject all V3 constructs with `WOLIO-FEATURE-001` and preserve bit-identical
`sym_` IDs and snapshot hashes, and optimize incremental module caching in
`wolio-lsp` and `wolio-format` for 100-module workspaces, completing M12.

## Authority
- Normative: [EXTENSION_BOUNDARIES.md](../02-specifications/EXTENSION_BOUNDARIES.md), [CORE_V3.md](../02-specifications/language/CORE_V3.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-REGRESSION`, M12 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Language version enum (`crates/wolio-diagnostics` / `wolio-ast`):**
  ```rust
  #[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash)]
  pub enum LanguageVersion {
      V1,
      V2,
      V3,
  }
  ```
- **Manifest version rules (`wolio.toml`):**
  - `[language] version = "1"` -> `LanguageVersion::V1` (rejects V2 and V3 constructs with `WOLIO-FEATURE-001`).
  - `[language] version = "2"` -> `LanguageVersion::V2` (enables `CORE_V2`, rejects V3 constructs `std::*`, `Secret<T>`, `resource`, `modifies`, `old`, `transaction`, `property`, `with_mock` with `WOLIO-FEATURE-001`).
  - `[language] version = "3"` -> `LanguageVersion::V3` (enables `CORE_V1` + `CORE_V2` + `CORE_V3`).
  - Any other version string (e.g., `"4"`, `"0"`) fails with `WOLIO-FEATURE-001` (exit code `1`).
- **Central V3 diagnostic registration (`crates/wolio-diagnostics`):**
  - Register `WOLIO-SECURITY-001`, `WOLIO-EFFECT-002`, `WOLIO-RESOURCE-001`, and `WOLIO-FUZZ-001` in the diagnostic registry (`EVIDENCE_MODEL.md`).
- **Incremental 100-module LSP & formatter profiling (`crates/wolio-semantic`, `crates/wolio-lsp`):**
  - Cache parsed ASTs and per-module symbol tables keyed by content SHA-256 so an incremental edit to 1 buffer in a 100-module synthetic V2 workspace only re-parses the edited buffer and re-checks its reverse transitive dependents, keeping incremental `textDocument/didChange` -> diagnostic publication latency `< 50 ms` and bounded memory usage (`< 256 MB` RSS).

## Scope
In scope:
- Extending `LanguageVersion` with `V3` and wiring `[language] version = "1" | "2" | "3"` across `wolio-cli`, `wolio-parser`, `wolio-semantic`, `wolio-types`, and `wolio-verify`.
- Registering V3 diagnostic codes (`WOLIO-SECURITY-001`, `WOLIO-EFFECT-002`, `WOLIO-RESOURCE-001`, `WOLIO-FUZZ-001`) in `wolio-diagnostics`.
- Per-module content-addressed AST/symbol caching in `wolio-lsp` / `wolio-semantic` and automated benchmark test over a 100-module synthetic workspace.

Out of scope (owner):
- Implementing `std::*` modules (T304, T305, T307, T308), `Secret<T>` (T306), `with_mock` (T309), `resource`/`transaction` (T310, T311), and `property` fuzzing (T312).

## Normative requirements
- Using any V3 construct (`import std::*`, `Secret<T>`, `resource`, `modifies`, `old`, `transaction`, `property`, `with_mock`) in a project declaring `[language] version = "1"` or `"2"` must fail deterministically with `WOLIO-FEATURE-001` (CORE_V3, EXTENSION_BOUNDARIES).
- Upgrading the compiler binary to V3 must keep `sym_` declaration IDs, clause hashes, and snapshot hashes bit-identical for all existing V1 (`version = "1"`) and V2 (`version = "2"`) projects (DELIVERY_POLICY).
- Incremental LSP updates on a 100-module V2 workspace must only recompute affected modules and preserve unsaved in-memory buffer isolation (V3-TASKS, DELIVERY_POLICY).

## Acceptance criteria
Positive and boundary:
- [ ] `wolio.toml` accepts `[language] version = "1"`, `"2"`, and `"3"`
- [ ] All V1 conformance projects (`version = "1"`) and V2 conformance projects (`version = "2"`) produce bit-identical `sym_` IDs, clause hashes, and snapshot hashes under the V3 compiler
- [ ] All valid V1 and V2 programs compile and run identically when upgraded to `[language] version = "3"`
- [ ] 100-module synthetic workspace benchmark completes single-file incremental LSP diagnostics and hover in `< 50 ms` without memory growth across 100 edits

Rejection and failure:
- [ ] Any V3 construct used under `[language] version = "1"` or `"2"` fails with `WOLIO-FEATURE-001`
- [ ] Unsupported `[language] version` values (e.g., `"4"`) fail with `WOLIO-FEATURE-001`

Completion:
- [ ] All M12 `DELIVERY_PLAN` and `ACCEPTANCE_MATRIX` (`V3-REGRESSION`) gates pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T303.md`

## Verification
```bash
cargo test -p wolio-parser -p wolio-semantic -p wolio-lsp
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T303.md` with V1/V2/V3 hash isolation proofs and 100-module LSP latency/memory benchmark numbers.

## Dependencies and open items
- Depends on: T302; blocks: T304 (and therefore M13).
- Open: none.

