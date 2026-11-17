`.blueprint/04-tasks/T318-foreign-contract-boundary-marshalling.md`
# T318: Enforce contract boundaries and error code marshalling on `foreign operation`s

| Field | Value |
|---|---|
| Milestone | `M14` |
| Priority | `P0` |
| Depends on | T317 |
| Blocks | T319 |
| Acceptance families | V3-FFI (contract boundaries and error marshalling) |
| Bounded areas | `crates/wolio-semantic` (submodules `contracts`, `effects`), `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T318.md` |

## Objective
Implement contract boundary enforcement and automatic C glue wrapper generation
for `foreign operation`s, evaluating `requires` preconditions before crossing the
C ABI boundary, marshalling C integer return codes and `errno` into structured
`Result<T, ForeignError>`, and evaluating `ensures` postconditions upon return,
ensuring AI vibe coders never interact with raw pointers or unchecked C errors.

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 7: Foreign Function Interface), [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.7: Foreign Function Interface & C glue generation)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-FFI`, M14 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Precondition evaluation before C boundary crossing (`crates/wolio-semantic::contracts`):**
  - When a `foreign operation` is called:
    1. The runtime evaluates `requires` precondition clauses in Wolio before any foreign code executes.
    2. If `requires` evaluates to `false`, execution immediately faults with `WOLIO-CONTRACT-002` (precondition violation); foreign C symbols are never invoked with invalid arguments.
    3. If an arithmetic overflow occurs during precondition evaluation, execution faults with `WOLIO-CONTRACT-004`.
- **C Return Code and Status Marshalling (`crates/wolio-runtime`):**
  - Standard foreign error enum:
    ```wol
    pub enum ForeignError {
        NullPointer,
        IoFailure,
        InvalidResponse,
        BufferTooSmall,
        PermissionDenied,
        Custom(Int)
    }
    ```
  - Return types of foreign operations must be `Result<T, ForeignError>` or `Result<Unit, ForeignError>`.
  - Compiler generates C glue translation:
    - Status return codes: C return `< 0` or non-zero status integer maps to `Result::Err(ForeignError::...)` based on standard mapping or errno inspection.
    - Pointer returns: C `NULL` pointer for operations returning non-optional handles maps to `Result::Err(ForeignError::NullPointer)`.
    - Success: return value is wrapped in `Result::Ok(val)` where `val` is translated into native Wolio types (fixed-width integers, boolean, or opaque foreign handle).
- **Postcondition evaluation after C return (`crates/wolio-semantic::contracts`):**
  - After foreign C code completes and return values are marshalled into `result: Result<T, ForeignError>`:
    1. The runtime evaluates `ensures` postcondition clauses over `result`.
    2. If `ensures` evaluates to `false`, execution immediately faults with `WOLIO-CONTRACT-003` (postcondition violation).
    3. This ensures that buggy or misbehaving C libraries cannot corrupt Wolio state or violate domain invariants undetected.
- **Zero raw pointer exposure:**
  - Neither AI coding agents nor human programmers can write raw pointer types (`*mut T`, `*const T`), dereference pointers, or construct `unsafe` blocks.
  - All FFI interactions are 100% typed, memory-safe, and contract-verified.

## Scope
In scope:
- Contract boundary wrapper generation in `wolio-semantic::contracts`.
- Pre-checking `requires` before C invocation and post-checking `ensures` after C return.
- `ForeignError` enum definition and return code marshalling in `wolio-runtime`.
- Runtime contract guard integration for foreign calls.

Out of scope (owner):
- Manifest validation and foreign syntax parsing (T316).
- Foreign type storage checking and fixed-width integer types (T317).
- Native LLVM IR lowering, linking, and test mocking (T319).

## Normative requirements
- Any foreign operation call whose arguments violate its `requires` clause must fault with `WOLIO-CONTRACT-002` without calling the foreign C symbol (CORE_V3, CONTRACT_DESIGN).
- Any foreign operation return that violates its `ensures` clause must fault with `WOLIO-CONTRACT-003` (CORE_V3, CONTRACT_DESIGN).
- Foreign operations must return `Result<T, ForeignError>` to enforce exhaustive error handling (`WOLIO-ERROR-001`, CORE_V3).

## Acceptance criteria
Positive and boundary:
- [ ] Calling a foreign operation with valid arguments executing a compliant C function returns `Result::Ok(val)` satisfying `ensures`
- [ ] C function returning negative status code correctly marshals to `Result::Err(ForeignError::...)` and satisfies error-branch `ensures`
- [ ] Fixed-width integer arguments (`UInt8`, `UInt16`, etc.) pass directly to C functions matching standard C integer ABI sizes

Rejection and failure:
- [ ] Calling a foreign operation with arguments violating `requires` faults with `WOLIO-CONTRACT-002` and does not invoke C glue
- [ ] A C function returning a value that violates `ensures` faults with `WOLIO-CONTRACT-003`
- [ ] Declaring a foreign operation returning bare `T` instead of `Result<T, ForeignError>` fails type checking with `WOLIO-TYPE-001`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T318.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T318.md` with `V3-FFI` contract pre/post guard and error code marshalling fixtures.

## Dependencies and open items
- Depends on: T317; blocks: T319.
- Open: none.

