`.blueprint/04-tasks/T051-llvm-lowering.md`
# T051: Pin LLVM toolchain and lower `VerifiedWolioIr` arithmetic and branches

| Field | Value |
|---|---|
| Milestone | `M05` |
| Priority | `P0` |
| Depends on | T050 |
| Blocks | T052 |
| Acceptance families | NATIVE (arithmetic, branches, target check) |
| Bounded areas | codegen (`crates/wolio-codegen`), toolchain/CI configuration |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T051.md` |

## Objective
Pin the exact LLVM version and build digest for M05 and lower `VerifiedWolioIr`
scalars, control flow, calls and checked 64-bit integer arithmetic to Linux
x86_64 object code.

## Authority
- Normative: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md), [CORE_V1.md](../02-specifications/language/CORE_V1.md), [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md)
- Decision: [ADR-007](../01-architecture/decisions/ADR-007-deterministic-identity-and-gated-automation.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `NATIVE`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Pinning the supported LLVM major version and exact build/container digest in
  repository toolchain/CI configuration without making LLVM a dependency of
  frontend/interpreter-only builds.
- `wolio-codegen` entry accepting only `VerifiedWolioIr`.
- LLVM lowering and Linux x86_64 object emission for `Int`, `Bool`, `Unit`,
  branches, short-circuit logic, operation calls and checked integer arithmetic.

Out of scope (owner):
- Heap values (records, enums, strings), `console::print` ABI and executable
  linking (T052); `wolio build` CLI and full differential release suite (T053).

## Normative requirements
- M05 pins a supported LLVM major version and exact build/container digest
  before backend implementation; LLVM is not a dependency of frontend or
  interpreter-only builds (DELIVERY_POLICY).
- Codegen accepts only `VerifiedWolioIr`; it cannot accept raw AST or unverified
  IR, nor repair or reinterpret language semantics (DELIVERY_POLICY, ADR-007).
- Checked 64-bit signed `Int` arithmetic never wraps in any build mode:
  overflow, division by zero and `MIN / -1` trap to the runtime fault path
  (`WOLIO-RUNTIME-001`); division truncates toward zero; remainder takes the
  dividend's sign; `MIN % -1` yields `0` (CORE_V1, INTERFACE_CONTRACTS,
  DELIVERY_POLICY).
- `&&`, `||`, `if` and `match` evaluate only the selected continuation
  (CORE_V1).
- Linux x86_64 is the V1 target; unsupported targets fail explicitly before
  code emission (DELIVERY_POLICY, ACCEPTANCE_MATRIX).

## Acceptance criteria
Positive and boundary:
- [ ] Pinned LLVM version and digest are recorded in toolchain/CI configuration; frontend/interpreter builds succeed without LLVM
- [ ] `VerifiedWolioIr` scalar arithmetic, comparisons, short-circuit logic, branches and calls lower to valid Linux x86_64 object code
- [ ] Boundary values (`MAX`, `MIN`, `MIN % -1`, truncating division) lower with exact CORE_V1 semantics

Rejection and failure:
- [ ] Codegen API cannot be invoked without `VerifiedWolioIr`
- [ ] Overflow, divide by zero and `MIN / -1` trigger the fault path even with LLVM optimizations enabled
- [ ] Unsupported target triples are rejected with a structured diagnostic

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T051.md` with pinned LLVM metadata and NATIVE arithmetic/branch fixtures.

## Dependencies and open items
- Depends on: T050; blocks: T052.
- Execution prerequisites: pinned LLVM development libraries/toolchain on the
  Linux x86_64 host (record a blocker under `.execution/blockers/` if
  unavailable).
- Open: none.

