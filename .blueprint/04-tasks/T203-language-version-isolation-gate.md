`.blueprint/04-tasks/T203-language-version-isolation-gate.md`
# T203: Enforce `[language] version = "1" | "2"` isolation gate

| Field | Value |
|---|---|
| Milestone | `M08` |
| Priority | `P0` |
| Depends on | T202 |
| Blocks | T204 (M09 entry) |
| Acceptance families | V2-REGRESSION; completes M08 |
| Bounded areas | `crates/wolio-cli`, `crates/wolio-parser`, `crates/wolio-semantic`, `crates/wolio-diagnostics` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T203.md` |

## Objective
Enable `[language] version = "2"` in `wolio.toml` and compiler configuration
while strictly isolating `[language] version = "1"` projects so they reject all
V2 constructs with `WOLIO-FEATURE-001` and preserve bit-identical `sym_` IDs,
clause hashes, snapshot hashes, and diagnostics, completing M08.

## Authority
- Normative: [EXTENSION_BOUNDARIES.md](../02-specifications/EXTENSION_BOUNDARIES.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md), [CORE_V2.md](../02-specifications/language/CORE_V2.md), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-REGRESSION`, M08 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Language version enum (`crates/wolio-diagnostics` or `wolio-ast`):**
  ```rust
  #[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash)]
  pub enum LanguageVersion {
      V1,
      V2,
  }
  ```
- **Manifest parsing (`wolio.toml`):**
  - `[language] version = "1"` -> `LanguageVersion::V1`
  - `[language] version = "2"` -> `LanguageVersion::V2`
  - `[language] version = "3"` (or any other value in V2 compiler) -> fails with `WOLIO-FEATURE-001` (exit code `1`).
- **Diagnostic registry registration (`crates/wolio-diagnostics`):**
  - Register V2 diagnostic codes in the central registry (`WOLIO-CONTRACT-005`, `WOLIO-CONTRACT-006`, `WOLIO-ARCH-001`) so code validation recognizes them in M09–M11.
- **Hash & identity stability guarantee:**
  - `sym_` hash domain remains `"wolio.symbol.v1"` across V1 and V2; upgrading a project from `version = "1"` to `version = "2"` without changing declarations preserves every `sym_` ID.

## Scope
In scope:
- Parsing and propagating `LanguageVersion::{V1, V2}` from `wolio.toml` (and standalone file default/flag) through `wolio-cli`, `wolio-parser`, `wolio-semantic`, `wolio-types`, and `wolio-verify`.
- Registering V2 diagnostic code metadata (`WOLIO-CONTRACT-005`, `WOLIO-CONTRACT-006`, `WOLIO-ARCH-001`) in `wolio-diagnostics`.
- Conformance tests proving that `version = "1"` projects produce bit-for-bit identical `sym_` IDs, snapshot hashes, and diagnostics under the V2 compiler, and that `version = "3"` or unknown versions fail with `WOLIO-FEATURE-001`.

Out of scope (owner):
- Parsing V2 `for` loops and `predicate` declarations (T204).
- Implementing V2 builtin scalar/collection types (T205, T207).

## Normative requirements
- Projects declaring `[language] version = "1"` must continue to enforce all V1 restrictions (`WOLIO-FEATURE-001` on V2 constructs) and produce identical `sym_` hashes, snapshot hashes, and diagnostics when compiled with a V2 compiler (BASELINE-V2, DELIVERY_POLICY, EXTENSION_BOUNDARIES).
- Projects declaring `[language] version = "2"` enable the `CORE_V2.md` profile; every valid V1 program compiles and executes identically under `version = "2"` (CORE_V2).
- Unsupported `[language] version` values (e.g., `"3"` or `"99"` in V2) fail deterministically with `WOLIO-FEATURE-001` (INTERFACE_CONTRACTS, DELIVERY_POLICY).

## Acceptance criteria
Positive and boundary:
- [ ] `wolio.toml` accepts both `[language] version = "1"` and `[language] version = "2"`
- [ ] All V1 conformance projects produce bit-identical `sym_` IDs, clause hashes, and snapshot hashes under `LanguageVersion::V1`
- [ ] All valid V1 conformance programs compile and run identically when their manifest is set to `[language] version = "2"`

Rejection and failure:
- [ ] `[language] version = "3"` or any unsupported version string fails with `WOLIO-FEATURE-001`
- [ ] Any V2-only feature gate checked under `LanguageVersion::V1` emits `WOLIO-FEATURE-001` (exit code `1`)

Completion:
- [ ] All M08 `DELIVERY_PLAN` and `ACCEPTANCE_MATRIX` (`V2-REGRESSION`) gates pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T203.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T203.md` with V1-vs-V2 snapshot/symbol hash parity checks and M08 gate completion summary.

## Dependencies and open items
- Depends on: T202; blocks: T204 (and therefore M09).
- Open: none.

