`.blueprint/04-tasks/T209-reusable-pure-predicates.md`
# T209: Implement reusable pure `predicate` declarations and contract calls

| Field | Value |
|---|---|
| Milestone | `M10` |
| Priority | `P0` |
| Depends on | T208 |
| Blocks | T210 |
| Acceptance families | V2-PRED |
| Bounded areas | `crates/wolio-semantic` (submodules `resolver`, `types`, `graph`, `contracts`), `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T209.md` |

## Objective
Implement reusable, guaranteed-pure, acyclic `predicate` declarations with
deterministic `sym_` identities (`kind = "predicate"`) and allow them to be
called inside `requires`, `ensures`, `bound`, `invariant`, and other `predicate`
bodies while keeping `operation` calls forbidden in contracts.

## Authority
- Normative: [CORE_V2.md](../02-specifications/language/CORE_V2.md) (Section 5: Pure `predicate` declarations), [CONTRACT_DESIGN.md](../02-specifications/CONTRACT_DESIGN.md), [SEMANTIC_MODEL.md](../02-specifications/SEMANTIC_MODEL.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-PRED`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Identity & Semantic Graph (`crates/wolio-semantic::graph`):**
  - Declaration ID hash domain: `"wolio.symbol.v1"`, `declaration_kind = "predicate"`, `symbol_path = predicate_name`.
  - Unlike `operation`, a `predicate` has no separate `implementation` block (`WOLIO-RESOLVE-001` if an `implementation` block targets a `predicate` name).
  - Semantic graph edges: `calls_predicate` (from contract clause or caller `predicate` to callee `predicate`), `uses_type` (for parameter types).
  - Impact analysis (`wolio impact`) traverses reverse `calls_predicate` edges so changing a public or private `predicate` reports all affected operations, contracts, and tests.
- **Resolution & Purity Rules (`crates/wolio-semantic::resolver`, `crates/wolio-semantic::contracts`):**
  - `predicate` declarations obey module visibility (`pub predicate` is importable across modules via `module::PredName`; private `predicate` is module-local; violations emit `WOLIO-RESOLVE-001`).
  - Parameter names may not be `"result"` or shadow visible bindings (`WOLIO-RESOLVE-001`).
  - Return type must be `Bool` (`WOLIO-TYPE-001` if any other return type is written, or `WOLIO-SYNTAX-001` at parse time).
  - A `predicate` body may contain literals, parameters, field accesses, nominal/record/enum constructors, pure builtin functions (`List::*`, `Map::*`, `Decimal::*`, `UInt::*`, `Bytes::*`, `format`), `if`/`else`, exhaustive `match`, and calls to visible `predicate` declarations.
  - Calling any `operation` (even a pure operation with `effects []`) or using a `for` loop inside a `predicate` body fails with `WOLIO-CONTRACT-001`.
  - Direct or indirect cycles in the `predicate` call graph fail with `WOLIO-RESOLVE-001`.
- **Runtime Evaluation (`crates/wolio-runtime`):**
  - Calling a `predicate` inside `requires`, `ensures`, `bound`, or `invariant` evaluates the predicate body under contract fault semantics: any arithmetic overflow or division by zero inside the called `predicate` aborts with `WOLIO-CONTRACT-004` and attributes the failure to the enclosing contract clause (`WOLIO-CONTRACT-002`, `003`, `005`, or `006` when the predicate returns `false`).

## Scope
In scope:
- Resolving `predicate` declarations, visibility, and acyclic call graphs (`WOLIO-RESOLVE-001`).
- Assigning `sym_` IDs (`kind = "predicate"`) and `calls_predicate` impact edges in `wolio-semantic`.
- Checking `predicate` purity (`WOLIO-CONTRACT-001` on operation calls or loops) and allowing `predicate` calls inside `requires`, `ensures`, `bound`, and `invariant`.
- Evaluating `predicate` calls in `wolio-runtime`.

Out of scope (owner):
- `[architecture]` module import policies (T210); Agent Protocol v2 `predicate` mutation payloads (T211); LLVM lowering of `predicate` declarations (T213).

## Normative requirements
- A `predicate` has no separate `implementation` block, has zero effects, returns `Bool`, and may call only other `predicate` declarations and builtin pure constructors/accessors (CORE_V2).
- Predicate call graphs must be strictly acyclic (`WOLIO-RESOLVE-001` on direct or mutual recursion) (CORE_V2).
- `requires`, `ensures`, `bound`, and `invariant` clauses may call visible `predicate` declarations; calling an `operation` inside a contract or `predicate` remains forbidden (`WOLIO-CONTRACT-001`) (CORE_V2, CONTRACT_DESIGN).
- Arithmetic faults inside a `predicate` called from a contract clause produce `WOLIO-CONTRACT-004` (CONTRACT_DESIGN, EVIDENCE_MODEL).

## Acceptance criteria
Positive and boundary:
- [ ] Local and imported `pub predicate` declarations resolve, receive deterministic `sym_` IDs (`kind = "predicate"`), and appear in `wolio inspect` and `wolio impact`
- [ ] `requires`, `ensures`, `bound`, and `invariant` clauses call single and nested acyclic `predicate` declarations and enforce runtime guards (`WOLIO-CONTRACT-002`, `003`, `005`, `006`)

Rejection and failure:
- [ ] Direct or indirect recursive `predicate` call cycles fail with `WOLIO-RESOLVE-001`
- [ ] Calling a private `predicate` from another module or writing an `implementation` block for a `predicate` fails with `WOLIO-RESOLVE-001`
- [ ] Calling an `operation` inside a `predicate` body or having a non-`Bool` body expression fails with `WOLIO-CONTRACT-001` / `WOLIO-TYPE-001`
- [ ] Arithmetic overflow inside a called `predicate` during contract checking aborts with `WOLIO-CONTRACT-004`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T209.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T209.md` with `V2-PRED` conformance fixtures.

## Dependencies and open items
- Depends on: T208; blocks: T210.
- Open: none.

