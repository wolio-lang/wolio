`.blueprint/04-tasks/T314-v3-ir-llvm-formatter-lsp-parity.md`
# T314: Complete V3 Native LLVM, `wolio fmt`, and `wolio lsp` parity

| Field | Value |
|---|---|
| Milestone | `M15` |
| Priority | `P0` |
| Depends on | T313 |
| Blocks | T315 |
| Acceptance families | V3-RELEASE, NATIVE, TOOL |
| Bounded areas | `crates/wolio-ir`, `crates/wolio-codegen`, `crates/wolio-runtime`, `crates/wolio-format`, `crates/wolio-lsp` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T314.md` |

## Objective
Lower all `CORE_V3` constructs (`std::*`, `Secret<T>`, `resource`, `modifies`,
`old(expr)` snapshots, and `transaction` rollback regions) to `VerifiedWolioIr`
and native LLVM codegen with 100% differential parity against the interpreter,
and complete canonical `wolio fmt` and `wolio lsp` support for all V3 syntax.

## Authority
- Normative: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) (Section 2 & Section 4), [CORE_V3.md](../02-specifications/language/CORE_V3.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 2 & Section 3)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-RELEASE`, `NATIVE`, and `TOOL`, M15 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Typed IR & `VerifiedWolioIr` verifier extensions (`crates/wolio-ir`):**
  - Add `IrType::SecretPtr` and `IrType::ResourcePtr(TypeId)`.
  - Add `IrInst::CaptureOld { slot: u32, expr_fn: IrFuncId, args: Vec<IrValue> }` emitted in the operation entry block immediately after `requires` guards and before any body instructions, and `IrInst::ReadOld { slot: u32 }` permitted only inside `ensures` guard blocks.
  - Add structured `TransactionRegion { handle: IrValue, begin_block, body_block, commit_block, rollback_block, merge_block }`:
    - `wolio_ir::verify_ir` verifies that every `Result::Err` edge and every fault unwind edge inside a `TransactionRegion` routes through `rollback_block` (`wolio_rt_tx_rollback`) before reaching `ensures` evaluation or exiting the operation, and every `Result::Ok` edge routes through `commit_block` (`wolio_rt_tx_commit`).
- **Native LLVM Codegen & Runtime ABI (`crates/wolio-codegen`, `crates/wolio-runtime`):**
  - Link all `wolio_rt_std_string_*`, `wolio_rt_std_math_*`, `wolio_rt_std_collections_*`, `wolio_rt_std_json_*`, `wolio_rt_std_crypto_*`, `wolio_rt_std_time_*`, `wolio_rt_std_fs_*`, `wolio_rt_std_net_*`, `wolio_rt_secret_*`, and `wolio_rt_tx_begin`/`commit`/`rollback` C ABI symbols into native builds (`wolio build`).
  - Keep all `requires`, `ensures` (including `old(...)`), `bound`, `invariant`, and transaction rollback guards active in release-optimized builds.
- **Canonical Formatter (`crates/wolio-format`):**
  - Format `import std::<module>;`, `pub resource <Name>;`, `modifies [h1, h2];`, `old(expr)`, `transaction (handle) { ... }`, `with_mock { ... } in { ... }`, and `property <name> for <op>(...) { where ...; ... }` with 4-space indentation, LF newlines, `//` comment preservation, and post-format `sym_` ID invariance check.
- **Language Server (`crates/wolio-lsp`):**
  - Hover, go-to-definition, find-references, real-time diagnostics (`WOLIO-SECURITY-001`, `WOLIO-EFFECT-002`, `WOLIO-RESOURCE-001`), and version-checked rename for `std::*` imports/calls, `property` declarations, `resource` parameters, `modifies` references, `old(...)`, and `with_mock` handlers.

## Scope
In scope:
- Lowering and verifying all V3 constructs in `wolio-ir` (`VerifiedWolioIr`) and `wolio-codegen` (LLVM).
- Differential parity tests comparing `wolio run` and `wolio build` across all `V3-STD`, `V3-SECRET`, `V3-IO`, and `V3-TX` conformance fixtures.
- Updating `wolio-format` and `wolio-lsp` for all `CORE_V3` syntax.

Out of scope (owner):
- Full W01–W15 comparative industrial benchmark and V3 release packaging (T315).

## Normative requirements
- `VerifiedWolioIr` rejects any `ReadOld` instruction outside an `ensures` guard block or any `TransactionRegion` that can exit on `Err`/fault without executing `rollback_block` (DELIVERY_POLICY, IMPLEMENTATION_SPECIFICATION_V3).
- Native binaries built via `wolio build` must match the interpreter 100% on stdout, return values, `Secret<T>` redaction, transaction rollback state, fault codes, and violated contract clause IDs across all V1, V2, and V3 conformance suites (DELIVERY_POLICY, ACCEPTANCE_MATRIX).
- `wolio fmt` must be idempotent, preserve comments, and preserve all `sym_` IDs (including `kind = "property"`) on V3 source files (DELIVERY_POLICY).

## Acceptance criteria
Positive and boundary:
- [ ] All valid `V3-STD`, `V3-SECRET`, `V3-IO`, and `V3-TX` programs compile via `wolio build` and produce identical outputs, rollbacks, and diagnostics to `wolio run`
- [ ] `wolio fmt` formats all `CORE_V3` constructs idempotently while preserving `//` comments and `sym_` IDs
- [ ] `wolio lsp` resolves hover, definition, references, and workspace rename across `property`, `resource`, `modifies`, `old(...)`, `with_mock`, and `std::*` symbols

Rejection and failure:
- [ ] Hand-crafted or mutated IR with an unguarded `TransactionRegion` exit or `ReadOld` in body blocks is rejected by `wolio_ir::verify_ir`
- [ ] `wolio fmt --check` exits `1` on unformatted V3 files without modifying disk files

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T314.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T314.md` with V3 interpreter-vs-native differential parity transcript, formatter idempotence checks, and LSP session logs.

## Dependencies and open items
- Depends on: T313; blocks: T315.
- Execution prerequisites: pinned LLVM toolchain.
- Open: none.

