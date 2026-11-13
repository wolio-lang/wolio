`.blueprint/04-tasks/T004-diagnostics-cli.md`
# T004: Add deterministic diagnostics and CLI `init`/`check`

| Field | Value |
|---|---|
| Milestone | `M01` |
| Priority | `P0` |
| Depends on | T003 |
| Blocks | T020 (M02 entry) |
| Acceptance families | FRONT (CLI cases); completes M01 |
| Bounded areas | CLI crate (`crates/wolio-cli`), shared diagnostic values, project discovery |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T004.md` |

## Objective
Expose project initialization (`wolio init`) and frontend checking (`wolio check`)
through the CLI and return canonical machine-readable JSON diagnostics and
`not_run` semantic evidence records.

## Authority
- Normative: [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md) (project inputs, CLI and output), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md), [DIAGNOSTIC_SPEC.md](../02-specifications/verification/DIAGNOSTIC_SPEC.md), [CORE_V1.md](../02-specifications/language/CORE_V1.md) (M01 profile)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `FRONT`, M01 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **JSON stdout envelope (`--json`):**
  ```json
  {
    "schema_version": 1,
    "command": "check",
    "status": "success",
    "diagnostics": [],
    "evidence": [
      {
        "id": "obl_semantic_m01",
        "symbol": null,
        "clause": "semantic_checks",
        "source_hash": "<sha256>",
        "tool_version": "0.1.0",
        "method": "static",
        "status": "not_run",
        "diagnostics": [],
        "tests": []
      }
    ],
    "result": {}
  }
  ```
- **`wolio init <dir>` template:** Creates `<dir>/wolio.toml` (with `[package]`, `[language] version = "1"`, `[entrypoint] module = "main"`, `operation = "main"`) and `<dir>/src/main.wol` (`module main; pub operation main() -> Unit { effects []; dependencies []; } implementation main { unit() }`). Refuses to overwrite existing files (exit `2`).
- **Exit codes:** `0` success, `1` source rejection (`WOLIO-SYNTAX-001` or `WOLIO-FEATURE-001`), `2` CLI usage/manifest/IO configuration error, `3` internal compiler error.

## Scope
In scope: `init` and `check` commands, project source discovery, JSON and human diagnostic output, exit codes, and `tests/conformance/front/` fixture runner.

Out of scope (owner):
- Module uniqueness, imports and resolution (T020); identity targets (T023); `run` (T024); `test`/`verify`/`inspect`/`impact` (T032–T033).

## Normative requirements
- Command availability at M01: `init`, `check`. All commands accept `--json`; project commands accept `--project <directory>`; `check [file]` also accepts one standalone source. Invoking later-milestone commands (`run`, `test`, `verify`, `inspect`, `impact`, `agent`, `build`, `fmt`, `lsp`, `add`, `install`, etc.) or `[language] version != "1"` produces a usage error (exit `2`) or `WOLIO-FEATURE-001` diagnostic (exit `1`), never a successful no-op (INTERFACE_CONTRACTS).
- Discovery: `wolio.toml`, `src/**/*.wol`, optional `tests/**/*.wol`; sorted normalized root-relative paths; no directory symlink following; excludes `.wolio/`, `.wolio-log/`, `.git/`, build output, and dependency caches (INTERFACE_CONTRACTS).
- Diagnostics follow `DIAGNOSTIC_SPEC` shape, sort by `(source.file, source.start, code, target)`; `target` is `null` before `T023` identity assignment (EVIDENCE_MODEL).
- M01 `check` reports unimplemented semantic obligations as `not_run` (CORE_V1).

## Acceptance criteria
- [ ] `wolio check` succeeds for valid input and reports semantic obligations `not_run`
- [ ] Invalid input returns deterministic `WOLIO-SYNTAX-001` code, byte span, and message
- [ ] CLI exit status distinguishes `0` (success), `1` (source error), and `2` (usage/config error)
- [ ] `wolio init` creates a valid project without overwrites; `check` supports both `--project` and standalone `[file]` with `--json`
- [ ] Unsupported commands or unsupported `[language] version` return a usage or `WOLIO-FEATURE-001` diagnostic, never success
- [ ] Project discovery order and exclusions (`.git/`, `.wolio/`, symlinks) are covered by fixtures
- [ ] All M01 DELIVERY_PLAN checks and `tests/conformance/front/` fixtures pass; report written

## Verification
```bash
cargo test --workspace
cargo run -p wolio-cli -- check tests/conformance/front/valid/empty.wol --json
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
```
Evidence: `.execution/reports/T004.md`

## Dependencies and open items
- Depends on: T003; blocks: T020 and therefore M02.
- Open: none.
