`.blueprint/04-tasks/T032-test-runner-evidence-verify.md`
# T032: Implement test discovery, runner, evidence model and `verify` CLI

| Field | Value |
|---|---|
| Milestone | `M03` |
| Priority | `P0` |
| Depends on | T031 |
| Blocks | T033 |
| Acceptance families | EVID |
| Bounded areas | verification (`crates/wolio-verify`), CLI (`crates/wolio-cli`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T032.md` |

## Objective
Discover and execute Wolio `test` declarations deterministically with contract
guards enabled, aggregate obligation evidence bound to the snapshot, and expose
`wolio test` and `wolio verify`.

## Authority
- Normative: [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md), [CORE_V1.md](../02-specifications/language/CORE_V1.md), [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `EVID`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Inclusion of `tests/**/*.wol` and in-module `test <name> for <op> { ... }`
  declarations in `test` and `verify` modes.
- Deterministic test execution order, per-test captured console output, and
  guard/fault reporting.
- Obligation evidence records (`static`, `runtime_guard`, `test`, `review`;
  `pass`, `fail`, `not_run`, `unknown`) bound to snapshot/declaration hashes.
- `wolio test` and `wolio verify` CLI commands in human and `--json` modes.
- Automated scalar boundary and property fuzzing mode (`wolio verify --fuzz`) to
  proactively defeat AI test-overfitting and hardcoded shortcuts on scalar operations.

Out of scope (owner):
- `inspect` and `impact` CLI commands and compatibility comparison (T033).
- M04 analyzer policy checks, review disposition ledger and `finalize` (T044).

## Normative requirements
- Test blocks must type-check as `Bool` and resolve their target operation;
  `true` passes, `false` or any runtime/contract fault fails. Source tests
  cannot catch or suppress contract violations (CORE_V1, INTERFACE_CONTRACTS).
- `test` executes in deterministic module/name order with isolated captured
  `console::print` output (CORE_V1, INTERFACE_CONTRACTS).
- Each obligation records `id`, `symbol`, `clause`, `source_hash`,
  `tool_version`, `method` (`static`, `runtime_guard`, `test`, `review`),
  `status` (`pass`, `fail`, `not_run`, `unknown`) and related diagnostic/test
  IDs (EVIDENCE_MODEL).
- An installed runtime guard is reported with method `runtime_guard`, never as
  universal static proof; test success applies only to recorded test inputs;
  unsupported checks remain `not_run` (EVIDENCE_MODEL).
- `verify` runs `check`, `test` and M03 evidence gates (including required tests
  for operations and reachable declared error/boundary paths), binding evidence
  to exact source, manifest, lockfile, compiler and policy hashes. Any changed
  input or delete/recreate incarnation invalidates prior evidence
  (CONTRACT_DESIGN, SEMANTIC_MODEL, EVIDENCE_MODEL).
- `wolio verify --fuzz` exercises contract-guarded operations against scalar
  boundary values (`Int::MIN`, `Int::MAX`, `-1`, `0`, `1`, inequality bounds) and
  seeded deterministic fuzz samples satisfying `requires`; postcondition breaches
  or runtime faults fail with `WOLIO-VERIFY-002` (DIAGNOSTIC_SPEC).

## Acceptance criteria
Positive and boundary:
- [ ] `wolio test` and `wolio verify` execute tests in deterministic module/name order
- [ ] Evidence distinguishes `static`, `runtime_guard`, `test` and `review` methods and `pass`/`fail`/`not_run`/`unknown` statuses
- [ ] Evidence is bound to `(symbol, declaration_hash, snapshot_hash)` and tool version
- [ ] `wolio verify --fuzz` passes for correct contract-bounded scalar operations

Rejection and failure:
- [ ] Any source, manifest, lockfile or policy edit — or delete/recreate at the same semantic key — invalidates old evidence
- [ ] Missing required boundary or reachable declared-error test coverage fails `verify`
- [ ] False test results and runtime or contract faults inside tests fail with exit code 1 and structured diagnostics
- [ ] Under `wolio verify --fuzz`, an operation violating `ensures` on boundary/fuzz inputs fails with `WOLIO-VERIFY-002`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p wolio-cli -- test --project <evid-fixture-project> --json
cargo run -p wolio-cli -- verify --project <evid-fixture-project> --json
```
Evidence: `.execution/reports/T032.md` with the EVID fixture list.

## Dependencies and open items
- Depends on: T031; blocks: T033.
- Open: none.

