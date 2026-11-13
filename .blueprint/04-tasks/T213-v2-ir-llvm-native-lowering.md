`.blueprint/04-tasks/T213-v2-ir-llvm-native-lowering.md`
# T213: Lower V2 bounded loops, collections, predicates and `Decimal` to `VerifiedWolioIr` and LLVM

| Field | Value |
|---|---|
| Milestone | `M11` |
| Priority | `P0` |
| Depends on | T212 |
| Blocks | T214 |
| Acceptance families | V2-NATIVE |
| Bounded areas | `crates/wolio-ir`, `crates/wolio-codegen`, `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T213.md` |

## Objective
Extend `crates/wolio-ir`, the `VerifiedWolioIr` structural verifier,
`crates/wolio-codegen` (LLVM backend), and the `crates/wolio-runtime` C ABI to
compile and execute all `CORE_V2` constructs natively with 100% differential
parity against the interpreter.

## Authority
- Normative: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) (Section 2: Interpreter-first and verified native lowering), [CORE_V2.md](../02-specifications/language/CORE_V2.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md) (Section 2 & Section 3)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-NATIVE` and `NATIVE`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Typed IR extensions (`crates/wolio-ir`):**
  - Add IR types: `IrType::UInt64`, `IrType::Decimal128` (`i128`), `IrType::BytesPtr`, `IrType::ListPtr(Box<IrType>)`, `IrType::MapPtr(Box<IrType>, Box<IrType>)`.
  - Add structured bounded loop IR construct:
    - `LoopRegion { source_len, bound_block, entry_invariant_block, header_block, body_block, step_invariant_block, exit_invariant_block, after_block }`.
    - The structural verifier (`wolio_ir::verify_ir`) rejects any loop region missing its `BoundGuard` (`WOLIO-CONTRACT-005`), entry/step/exit `InvariantGuard` (`WOLIO-CONTRACT-006`), or monotonic index increment `idx_next = idx + 1` bounded by `source_len`. Unstructured backward jumps remain rejected.
  - Lower `predicate` declarations as pure IR functions callable from contract guard blocks and other predicates.
- **Runtime C ABI additions (`crates/wolio-runtime`):**
  - `wolio_rt_decimal_mul(a_lo: u64, a_hi: i64, b_lo: u64, b_hi: i64, ctx: *const FaultCtx) -> i128`
  - `wolio_rt_decimal_div(a_lo: u64, a_hi: i64, b_lo: u64, b_hi: i64, ctx: *const FaultCtx) -> i128`
  - `wolio_rt_bytes_empty()`, `wolio_rt_bytes_from_utf8(...)`, `wolio_rt_bytes_len(...)`, `wolio_rt_bytes_eq(...)`
  - `wolio_rt_list_empty()`, `wolio_rt_list_push(...)`, `wolio_rt_list_len(...)`, `wolio_rt_list_get(...)`, `wolio_rt_list_eq(...)`
  - `wolio_rt_map_empty()`, `wolio_rt_map_insert(...)`, `wolio_rt_map_get(...)`, `wolio_rt_map_remove(...)`, `wolio_rt_map_len(...)`, `wolio_rt_map_keys(...)`, `wolio_rt_map_eq(...)`
  - `wolio_rt_format(...)`
  - `wolio_rt_contract_fail_bound(ctx: *const ContractFaultCtx) -> !` (emits `WOLIO-CONTRACT-005`)
  - `wolio_rt_contract_fail_invariant(ctx: *const ContractFaultCtx) -> !` (emits `WOLIO-CONTRACT-006`)
- **LLVM Codegen (`crates/wolio-codegen`):**
  - Lower checked `UInt64` and `Decimal128` (`i128` add/sub overflow intrinsics + C ABI mul/div calls), bounded loop basic blocks with phi nodes for `(idx, acc)`, `predicate` calls, and reference-counted `List`/`Map`/`Bytes`/`format` calls.
  - Keep all `bound`, `invariant`, `requires`, `ensures`, and arithmetic guards active in release builds (`wolio build`).

## Scope
In scope:
- Lowering all `CORE_V2` constructs to `wolio-ir` and verifying them in `verify_ir` to produce `VerifiedWolioIr`.
- Extending the `wolio-runtime` C ABI and `wolio-codegen` LLVM backend for `UInt`, `Decimal`, `Bytes`, `List<T>`, `Map<K, V>`, `format(...)`, `predicate`, and bounded `for` loops.
- Differential parity test suite comparing `wolio run` (interpreter) and `wolio build` (native binary) across all `V2-LOOP`, `V2-DATA`, `V2-PRED`, and `V2-ARCH` conformance fixtures.

Out of scope (owner):
- V2 `wolio fmt` and `wolio lsp` updates (T214); 24-task V2 benchmark (T215); V2 release archive (T216); copy-on-write collection optimization (V3 `T302`).

## Normative requirements
- `VerifiedWolioIr` remains the sole entry into `wolio-codegen`; any loop IR missing `bound` or entry/step/exit `invariant` guard blocks or using unstructured backward branches is rejected by `verify_ir` (DELIVERY_POLICY, IMPLEMENTATION_SPECIFICATION_V2).
- Release-optimized native binaries produced by `wolio build` must retain all runtime arithmetic (`WOLIO-RUNTIME-001`), contract (`WOLIO-CONTRACT-002`..`004`), loop `bound` (`WOLIO-CONTRACT-005`), and loop `invariant` (`WOLIO-CONTRACT-006`) guards (DELIVERY_POLICY, CORE_V2).
- Interpreter and native execution must agree 100% on stdout, return values, fault categories, and violated contract/loop clause identities across all V1 and V2 conformance fixtures (DELIVERY_POLICY, ACCEPTANCE_MATRIX).

## Acceptance criteria
Positive and boundary:
- [ ] All valid `V2-LOOP`, `V2-DATA`, and `V2-PRED` programs lower to `VerifiedWolioIr`, compile via `wolio build`, and produce identical output to `wolio run`
- [ ] 128-bit `Decimal`, `UInt`, `List<T>`, `Map<K, V>` canonical key order, and `format(...)` behave identically in native and interpreter execution

Rejection and failure:
- [ ] Mutated or hand-crafted IR with an unbounded loop or missing `BoundGuard`/`InvariantGuard` is rejected by `wolio_ir::verify_ir`
- [ ] Native binaries abort with exact `WOLIO-CONTRACT-005` on false `bound`, `WOLIO-CONTRACT-006` on false entry/step/exit `invariant`, `WOLIO-CONTRACT-004` on predicate overflow, and `WOLIO-RUNTIME-001` on `UInt`/`Decimal` arithmetic faults

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T213.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T213.md` with interpreter-vs-native differential parity transcript across all V2 conformance suites.

## Dependencies and open items
- Depends on: T212; blocks: T214.
- Execution prerequisites: pinned LLVM toolchain from M05.
- Open: none.

