`.blueprint/04-tasks/T310-linear-resource-modifies-old-snapshots.md`
# T310: Implement linear `resource` handles, `modifies [...]` frames, and `old(expr)` snapshots (`WOLIO-RESOURCE-001`)

| Field | Value |
|---|---|
| Milestone | `M14` |
| Priority | `P0` |
| Depends on | T309 |
| Blocks | T311 |
| Acceptance families | V3-TX (`resource`, `modifies`, `old(...)`, and `WOLIO-RESOURCE-001`) |
| Bounded areas | `crates/wolio-ast`, `crates/wolio-parser`, `crates/wolio-semantic` (submodules `resolver`, `types`, `contracts`, `effects`, `graph`), `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T310.md` |

## Objective
Implement linear non-aliasing `resource <Type>` parameters, resolve resource
instantiation deadlocks via explicit scope-bound acquisition (`Resource::acquire::<T>`)
and test mocks (`Resource::mock::<T>`), enforce explicit `modifies [handle, ...]`
contract frame clauses (`WOLIO-RESOURCE-001`, Taxonomy `W09`), and capture pre-state
`old(expr)` snapshots in `ensures` postconditions.

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 5: Stateful resource handles, instantiation, and `transaction` blocks), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md), [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md) (`W09`)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.5: Linear `resource` handles, acquisition, `modifies [...]`, `old(expr)`)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-TX`, M14 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Grammar and AST (`crates/wolio-ast`, `crates/wolio-parser`):**
  - Parameter syntax: `ident, ":", [ "resource" ], type` (`ParamMode::Resource` when `"resource"` is present).
  - Operation contract clause: `"modifies", "[", [ ident, { ",", ident } ], "]", ";"` (`ContractClause::Modifies(Vec<Spanned<String>>)`).
  - Postcondition snapshot expression: `"old", "(", expr, ")"` (`Expr::Old(Box<Expr>)`).
  - Resource declaration: `pub resource DbHandle;` (declares a nominal linear resource handle type backed by a key-value/table state store in `wolio-runtime`).
- **Scope-bound lexical acquisition (deadlock resolution):**
  - **Production acquisition:** `Resource::acquire::<T>(config)` can be called only in package `main` or authorized capability adapter operations. Raw value or literal instantiation of resources is forbidden.
  - **Test mock handle:** `Resource::mock::<T>()` can be called only within an enclosing `with_mock` block in `test` and `property` declarations.
- **Linear non-aliasing and `modifies` rules (`crates/wolio-semantic::types`, `crates/wolio-semantic::effects` — `WOLIO-RESOURCE-001`):**
  1. **No aliasing in calls (`WOLIO-RESOURCE-001`):** Passing the same `resource` binding in two or more argument positions of a single operation or predicate call (e.g., `Sync(db, db)`) fails statically with `WOLIO-RESOURCE-001`.
  2. **No aliased rebinding (`WOLIO-RESOURCE-001`):** `let db2 = db;` to create a second live alias in the same scope fails with `WOLIO-RESOURCE-001`. Moving a resource invalidates the previous identifier.
  3. **No escaping or embedding (`WOLIO-RESOURCE-001`):** Storing a `resource` handle inside a `record`, `enum`, `Option`, `Result`, `List`, or `Map`, or returning a `resource` from an operation without explicit linear transfer capability fails with `WOLIO-RESOURCE-001`.
  4. **Mandatory `modifies [handle]` frame (`WOLIO-RESOURCE-001`):**
     - Every identifier in `modifies [h1, ...]` must be a `resource` parameter of that operation (`WOLIO-RESOURCE-001` otherwise).
     - Any operation that calls a mutating primitive on `handle` (e.g., `Resource::set(db, key, val)`) or passes `handle` to a callee operation whose `modifies` clause includes that parameter position **must** declare `handle` in its own `modifies [...]` clause and declare effect `Db::Tx` (or resource mutation effect); omitting `modifies [handle];` fails with `WOLIO-RESOURCE-001`.
     - Conversely, declaring `modifies [handle];` when the implementation never mutates `handle` (directly or transitively) fails with `WOLIO-ANALYZER-003` (unused frame padding).
- **`old(expr)` rules and runtime evaluation (`crates/wolio-semantic::contracts`, `crates/wolio-runtime`):**
  - `old(expr)` is permitted **only** inside `ensures` clauses (`WOLIO-CONTRACT-001` if used in `requires`, `bound`, `invariant`, `predicate`, or implementation bodies).
  - `expr` inside `old(expr)` must be a pure expression/predicate referencing at least one `resource` parameter declared in `modifies [...]`, must not reference `result`, and must not contain nested `old(...)` (`WOLIO-CONTRACT-001`).
  - At runtime, after `requires` succeeds and **before** the operation body executes, `wolio-runtime` evaluates each `old(expr)` expression against the entry state of the `resource` handle and stores the resulting value in an entry snapshot map keyed by AST node ID; when `ensures` runs after the body completes, `old(expr)` reads the captured entry value.

## Scope
In scope:
- Parsing and AST nodes for `resource` type declarations, `resource` parameters, `modifies [...]` contract clauses, and `old(expr)`.
- Static linear non-aliasing checks and transitive `modifies [...]` frame verification emitting `WOLIO-RESOURCE-001`.
- Entry snapshot evaluation of `old(expr)` in `wolio-contracts` and `wolio-runtime`.
-Including `modifies [...]` in semantic contract hashes and `wolio impact` (`HUMAN_REQUIRED` when changing `modifies` on a public/existing operation).

Out of scope (owner):
- `transaction (handle) { ... }` atomic rollback execution on `Result::Err` or fault (T311).

## Normative requirements
- Passing the same `resource` handle twice in one call (`f(db, db)`), aliasing it in a local `let`, or storing it in a composite type is rejected statically with `WOLIO-RESOURCE-001` (CORE_V3, EVIDENCE_MODEL, AI_BUG_TAXONOMY `W09`).
- Operations mutating a `resource` handle directly or transitively must declare `modifies [handle];` (`WOLIO-RESOURCE-001` if omitted) and `effects [Db::Tx]` (`WOLIO-EFFECT-001` if omitted) (CORE_V3, V3-TASKS).
- `old(expr)` in `ensures` is evaluated once at operation entry before the body executes and binds the pre-state value for postcondition evaluation (CORE_V3, V3-TASKS).

## Acceptance criteria
Positive and boundary:
- [ ] Operations taking `db: resource DbHandle` with `modifies [db];` and `ensures ... == old(AccountBalance(db, from)) - amount` capture the pre-state value at entry and verify the postcondition on exit
- [ ] Read-only inspector `predicate`s (e.g., `AccountBalance(db, id)`) can inspect `resource` state inside `requires`, `ensures`, and `old(...)` without mutating it
- [ ] Passing two distinct resource handles (`Migrate(source_db, target_db)`) with `modifies [target_db];` passes static checking

Rejection and failure:
- [ ] Passing the same resource binding twice (`TransferBetween(db, db)`) or aliasing `let db2 = db;` fails with `WOLIO-RESOURCE-001`
- [ ] Mutating a `resource` handle directly or via a helper operation without declaring `modifies [db];` fails with `WOLIO-RESOURCE-001`
- [ ] Using `old(expr)` outside `ensures`, nesting `old(old(x))`, or referencing `result` inside `old(...)` fails with `WOLIO-CONTRACT-001`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T310.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T310.md` with `V3-TX` linear resource, `modifies`, and `old(expr)` conformance fixtures.

## Dependencies and open items
- Depends on: T309; blocks: T311.
- Open: none.

