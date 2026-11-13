`.blueprint/04-tasks/T050-typed-ir-verifier.md`
# T050: Implement typed IR lowering and the `VerifiedWolioIr` structural verifier

| Field | Value |
|---|---|
| Milestone | `M05` |
| Priority | `P0` |
| Depends on | T034 |
| Blocks | T051 |
| Acceptance families | NATIVE (invalid IR and verified boundary) |
| Bounded areas | IR (`crates/wolio-ir`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T050.md` |

## Objective
Lower checked Wolio semantics into a typed SSA-like intermediate representation
with explicit arithmetic and contract guards, enforced by a structural verifier
that alone constructs `VerifiedWolioIr`.

## Authority
- Normative: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md), [CORE_V1.md](../02-specifications/language/CORE_V1.md), [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md), [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)
- Decision: [ADR-007](../01-architecture/decisions/ADR-007-deterministic-identity-and-gated-automation.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `NATIVE`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Typed SSA-like IR data structures covering blocks, branches, calls,
  constructors, tags, checked arithmetic guards and `requires`/`ensures`
  contract guards.
- Lowering from the checked M03 semantic representation into Wolio IR.
- Structural IR verifier validating types, control-flow edges, terminators and
  guard invariants, returning the opaque `VerifiedWolioIr` token bound to the
  snapshot hash.

Out of scope (owner):
- LLVM toolchain pinning and machine/object emission (T051); native runtime ABI
  and linking (T052); `wolio build` CLI and differential parity suite (T053).

## Normative requirements
- M05 lowers checked semantics into typed SSA-like IR with explicit basic
  blocks, branches, calls, constructors, variant tags, arithmetic fault guards
  and contract `requires`/`ensures` guards (DELIVERY_POLICY).
- The structural IR verifier rejects ill-typed values or edges, invalid tag/field
  accesses, missing or malformed block terminators, and missing arithmetic or
  contract guard instructions (DELIVERY_POLICY, ACCEPTANCE_MATRIX).
- Only the IR verifier may construct the opaque `VerifiedWolioIr` value accepted
  by native codegen. `VerifiedWolioIr` binds immutable IR to the snapshot hash
  and completed static checks; any IR transformation requires re-verification
  (DELIVERY_POLICY, ADR-007).
- `wolio-ir` has no dependency on LLVM or `wolio-codegen` (DELIVERY_POLICY).

## Acceptance criteria
Positive and boundary:
- [ ] All valid M01–M03 conformance programs lower to Wolio IR and pass the verifier to produce `VerifiedWolioIr`
- [ ] Arithmetic fault guards and source-ordered `requires`/`ensures` guards are explicit in IR
- [ ] `VerifiedWolioIr` cannot be constructed outside the verifier module and binds to the snapshot hash

Rejection and failure:
- [ ] Hand-crafted or mutated IR with ill-typed edges, mismatched branch types, bad tags or missing terminators is rejected by the verifier
- [ ] IR missing required arithmetic or contract guard nodes is rejected

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
Evidence: `.execution/reports/T050.md` with valid/invalid IR verifier fixtures.

## Dependencies and open items
- Depends on: T034; blocks: T051.
- Open: none.

