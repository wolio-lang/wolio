`.blueprint/04-tasks/T319-foreign-llvm-lowering-with-mock.md`
# T319: Lower foreign bindings and fixed-width integers to textual LLVM IR, and enforce `with_mock` test sandbox

| Field | Value |
|---|---|
| Milestone | `M14` |
| Priority | `P0` |
| Depends on | T318 |
| Blocks | T312 (M15 entry) |
| Acceptance families | V3-FFI, NATIVE; completes M14 |
| Bounded areas | `crates/wolio-ir`, `crates/wolio-codegen`, `crates/wolio-verify` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T319.md` |

## Objective
Lower foreign function declarations, contract boundary wrappers, scope-bound
destructor glue, and fixed-width integer scalar types (`i8`, `i16`, `i32`) to
verified textual LLVM IR (`.ll`), compile and link against authorized C libraries
via pinned `clang`, and enforce hermetic `with_mock` test sandbox coverage for
all operations with effect `Foreign::Call` (`WOLIO-EFFECT-002`), completing
milestone M14.

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 7: Foreign Function Interface), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md), [BASELINE-V3.md](../BASELINE-V3.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.7 & Section 3.8)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-FFI` and `NATIVE`, M14 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Verified IR extensions (`crates/wolio-ir`):**
  - Add `IrType::ForeignPtr(TypeId)` and fixed-width integer types (`IrType::UInt8`, `IrType::UInt16`, `IrType::UInt32`, `IrType::Int8`, `IrType::Int16`, `IrType::Int32`).
  - Add `IrInst::ForeignCall { symbol: String, library: String, args: Vec<IrValue>, ret_ty: IrType }`.
  - Add `IrInst::ForeignDrop { handle: IrValue, drop_symbol: String }` inserted at lexical block exit terminators.
- **Textual LLVM IR lowering (`crates/wolio-codegen`):**
  - Emits `declare` statements for foreign C symbols with standard C calling conventions:
    ```llvm
    declare i32 @hardware_read_reg(i8* %handle, i16 %offset, i8* %out_val)
    declare void @hardware_device_drop(i8* %handle)
    ```
  - Maps fixed-width integer types natively:
    - `UInt8` / `Int8` -> `i8`
    - `UInt16` / `Int16` -> `i16`
    - `UInt32` / `Int32` -> `i32`
  - Generates LLVM IR wrappers that:
    1. Evaluate `requires` basic blocks and branch to `@wolio_rt_contract_fault` on false.
    2. Invoke the declared extern C symbol.
    3. Check C status code / NULL pointers and construct `Result<T, ForeignError>` in memory.
    4. Evaluate `ensures` basic blocks over the resulting `Result` before returning.
- **Toolchain linking via pinned `clang`:**
  - Passes authorized libraries from `wolio.toml [foreign] allowed_libraries` as linker flags (e.g., `-lhardware -lsqlite3`) during `wolio build`.
  - Linking against any library not listed in `allowed_libraries` is strictly prevented by `wolio-codegen`.
- **Hermetic test runner sandbox with `with_mock` (`crates/wolio-verify`):**
  - Calling any operation declaring `effects [Foreign::Call]` inside a `test` or `property` declaration requires an enclosing `with_mock` handler table covering that operation:
    ```wol
    with_mock {
        ReadRegister => fn(dev: DeviceHandle, offset: UInt16) -> Result<UInt8, ForeignError> {
            Result::Ok(UInt8::from_uint(42))
        }
    } in {
        let dev = DeviceHandle::mock();
        let res = ReadRegister(dev, UInt16::from_uint(10));
        test::assert(res == Result::Ok(UInt8::from_uint(42)));
    }
    ```
  - Unmocked foreign calls in `wolio test` or `wolio verify` fail immediately with diagnostic `WOLIO-EFFECT-002`.

## Scope
In scope:
- Textual LLVM IR `.ll` emission for foreign declarations and wrappers in `wolio-codegen`.
- Native `i8`, `i16`, `i32` LLVM type lowering and checked arithmetic.
- Linker invocation via pinned `clang` using authorized libraries.
- Hermetic `with_mock` verification and unmocked call rejection (`WOLIO-EFFECT-002`) in `wolio-verify`.

Out of scope (owner):
- Property fuzzing counterexample generation (T312).
- Agent Protocol v3 mutation schemas (T313).
- Full W01–W16 comparative industrial benchmark (T315).

## Normative requirements
- `wolio-codegen` must emit verified textual LLVM IR (`.ll`) and invoke `clang`; it must not use `llvm-sys` or `inkwell` C++ bindings (DELIVERY_POLICY).
- External libraries passed to `clang` must strictly match `wolio.toml [foreign] allowed_libraries` (CORE_V3, INTERFACE_CONTRACTS).
- Live foreign calls during `wolio test` or `wolio verify` without a mock handler must fail with `WOLIO-EFFECT-002` (CORE_V3, DIAGNOSTIC_SPEC).

## Acceptance criteria
Positive and boundary:
- [ ] Foreign operation lowers to valid textual LLVM IR with `declare` extern C symbol and compiles successfully with `clang`
- [ ] Fixed-width integers lower to LLVM `i8`, `i16`, `i32` instructions and execute with checked arithmetic parity against the interpreter
- [ ] Tests mocking foreign operations via `with_mock` pass in `wolio test` without invoking live C libraries
- [ ] Scope exit in LLVM-compiled binary invokes auto-drop destructor glue without memory leaks

Rejection and failure:
- [ ] Executing a test that calls an operation with `effects [Foreign::Call]` without `with_mock` fails with `WOLIO-EFFECT-002`
- [ ] Linking fails if foreign library is missing or symbol is not resolved by `clang`

Completion:
- [ ] All M14 `DELIVERY_PLAN` and `ACCEPTANCE_MATRIX` (`V3-IO`, `V3-TX`, `V3-FFI`) gates pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T319.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T319.md` with `V3-FFI` LLVM lowering, linking, and `with_mock` sandbox conformance fixtures.

## Dependencies and open items
- Depends on: T318; blocks: T312 (and therefore M15).
- Open: none.

