`.blueprint/04-tasks/T052-runtime-abi-linking.md`
# T052: Document C/runtime ABI and link records, enums, strings, console and guards

| Field | Value |
|---|---|
| Milestone | `M05` |
| Priority | `P0` |
| Depends on | T051 |
| Blocks | T053 |
| Acceptance families | NATIVE (records, enums, strings, console, guards) |
| Bounded areas | runtime (`crates/wolio-runtime`), codegen (`crates/wolio-codegen`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T052.md` |

## Objective
Define the native runtime ABI and implement lowering and static linking for
nominal types, records, tagged enums, reference-counted UTF-8 strings,
recursive equality, `console::print` and contract/arithmetic fault handlers.

## Authority
- Normative: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md), [CORE_V1.md](../02-specifications/language/CORE_V1.md), [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `NATIVE`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Written specification of the V1 native C/runtime ABI (value representations,
  calling conventions, reference-count lifecycle, fault/guard entrypoints).
- Lowering and runtime support for nominal wrappers, records, enums (`Option`,
  `Result`, user enums), UTF-8 strings and recursive structural equality.
- Runtime adapter for `console::print(String) -> Unit` and structured abort
  reporting for `WOLIO-RUNTIME-001` and `WOLIO-CONTRACT-002`/`003`/`004`.
- Linking emitted object files with the native runtime library into runnable
  Linux x86_64 binaries.

Out of scope (owner):
- `wolio build` CLI command and full release differential parity gate (T053).

## Normative requirements
- `wolio-runtime` never depends on `wolio-codegen` or LLVM (DELIVERY_POLICY).
- Immutable heap values use reference counting initially, with value semantics
  and no exposed pointer identity (CORE_V1, DELIVERY_POLICY).
- Recursive structural equality works across scalars, nominals, records and
  enums; `console::print(String) -> Unit` is the sole host adapter (CORE_V1,
  DELIVERY_POLICY).
- Contract guards (`requires`, `ensures`) and arithmetic fault handlers remain
  active in native binaries, aborting on failure with the exact diagnostic code
  (`WOLIO-CONTRACT-002`, `003`, `004`, `WOLIO-RUNTIME-001`), clause identity,
  operation, caller and source span, without dumping runtime argument values
  (CONTRACT_DESIGN, EVIDENCE_MODEL, DELIVERY_POLICY).

## Acceptance criteria
Positive and boundary:
- [ ] Native ABI is documented and tested for scalars, records, tagged enums and reference-counted UTF-8 strings
- [ ] Recursive structural equality and `match` over enums/`Option`/`Result`/`Bool` execute natively
- [ ] `console::print` output and `Unit`/`Int` entrypoint return values match interpreter semantics

Rejection and failure:
- [ ] Native `requires`, `ensures` and predicate-overflow failures abort with `WOLIO-CONTRACT-002`/`003`/`004` and identify the exact clause and caller
- [ ] Reference-count lifecycle tests show no use-after-free or leak on normal return or early contract/arithmetic abort

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
Evidence: `.execution/reports/T052.md` with ABI documentation path and NATIVE composite/guard fixtures.

## Dependencies and open items
- Depends on: T051; blocks: T053.
- Execution prerequisites: pinned LLVM and Linux x86_64 linker toolchain.
- Open: none.

