`.blueprint/04-tasks/T309-hermetic-test-with-mock-sandbox.md`
# T309: Enforce hermetic test sandbox and mandatory `with_mock` handlers (`WOLIO-EFFECT-002`)

| Field | Value |
|---|---|
| Milestone | `M14` |
| Priority | `P0` |
| Depends on | T308 |
| Blocks | T310 |
| Acceptance families | V3-IO (`with_mock` and `WOLIO-EFFECT-002`) |
| Bounded areas | `crates/wolio-lexer`, `crates/wolio-ast`, `crates/wolio-parser`, `crates/wolio-semantic` (submodules `types`, `effects`), `crates/wolio-verify`, `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T309.md` |

## Objective
Implement `with_mock { <effect_op> => <pure_handler>, ... } in { <body> }` and
enforce a hermetic test and property sandbox that rejects any unmocked
non-console external I/O or foreign effect (`Clock::Read`, `Fs::*`, `Net::*`, `Db::*`,
`Foreign::Call`) in `wolio test` and `wolio verify` with `WOLIO-EFFECT-002` (Taxonomy `W13`).

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 4: Mandatory deterministic test mocks), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md), [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md) (`W13`)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.4: Hermetic test and property sandbox)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-IO`, M14 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Grammar and AST (`crates/wolio-lexer`, `crates/wolio-ast`, `crates/wolio-parser`):**
  ```text
  with_mock_expr = "with_mock", "{", mock_arm, { ",", mock_arm }, [ "," ], "}", "in", "{", { let_stmt }, expr, "}" ;
  mock_arm       = qualified_ident, "=>", "fn", "(", [ fields ], ")", "->", type, "{", { let_stmt }, expr, "}" ;
  ```
  ```rust
  pub struct MockHandlerAst {
      pub span: Span,
      pub target_op: Spanned<String>,
      pub params: Vec<FieldDecl>,
      pub return_type: TypeAst,
      pub lets: Vec<LetStmt>,
      pub body: Box<Expr>,
  }

  pub struct WithMockExpr {
      pub span: Span,
      pub handlers: Vec<MockHandlerAst>,
      pub lets: Vec<LetStmt>,
      pub body: Box<Expr>,
  }
  ```
- **Static typing and sandbox rules (`crates/wolio-semantic::types`, `crates/wolio-semantic::effects`):**
  - `with_mock` is permitted **only** inside `test` and `property` declarations; using `with_mock` inside a production `operation` or `predicate` fails with `WOLIO-EFFECT-002`.
  - Each `target_op` in `handlers` must resolve to an effectful operation (such as `std::time::now`, `std::fs::read_text`, `std::fs::write_text`, `std::net::http_send`, a foreign operation, or a user operation declaring `Db::*` / external effects), and its parameter types and return type must exactly match `target_op` (`WOLIO-TYPE-001`). Duplicate `target_op` entries in the same `with_mock` block fail with `WOLIO-SYNTAX-001` / `WOLIO-RESOLVE-001`.
  - Each mock handler body must be pure (`effects []`, no operation calls with external effects; `WOLIO-EFFECT-002` otherwise).
  - Inside a `test` or `property` declaration, every call to an operation `Op` whose transitive effects include any effect in `{Clock::Read, Fs::Read, Fs::Write, Net::Http, Db::Query, Db::Tx, Foreign::Call}` must be dynamically and statically covered by enclosing `with_mock` handlers for all leaf external operations producing those effects. Any uncovered external effect in a `test` or `property` fails static checking (`wolio check` / `wolio test` / `wolio verify`) with `WOLIO-EFFECT-002`.
- **Runtime dispatch (`crates/wolio-runtime`):**
  - `wolio-runtime` maintains a scoped mock handler stack during `test` and `property` evaluation. When a mocked operation is invoked, the runtime evaluates its `requires` guard, executes the pure mock handler instead of the live host adapter, and evaluates the operation's `ensures` guard on the mocked return value.
  - As a defense-in-depth runtime guard, the live host adapters (`std::time::now`, `std::fs::*`, `std::net::*`) check `ExecutionMode::TestSandbox` and immediately abort with `WOLIO-EFFECT-002` if ever reached without a mock handler.

## Scope
In scope:
- Lexing, parsing, and AST representation of `with_mock { ... } in { ... }`.
- Static type checking of mock handler signatures and transitive effect coverage analysis in `test`/`property` blocks (`WOLIO-EFFECT-002`).
- Scoped mock dispatch table and hermetic sandbox enforcement in `wolio-runtime` and `wolio-verify`.

Out of scope (owner):
- Linear `resource` handles, `modifies [...]`, and `old(expr)` snapshots (T310); `transaction` blocks (T311).

## Normative requirements
- Zero live external I/O (`Clock::Read`, `Fs::Read`, `Fs::Write`, `Net::Http`, `Db::Query`, `Db::Tx`) is permitted inside `wolio test` or `wolio verify`; any `test` or `property` invoking an operation with unmocked non-console effects fails with `WOLIO-EFFECT-002` (CORE_V3, EVIDENCE_MODEL, V3-TASKS).
- Mock handlers in `with_mock` must be pure deterministic expressions matching the target operation's exact parameter and return types, and `requires`/`ensures` guards on the mocked operation remain active during mock execution (CORE_V3, IMPLEMENTATION_SPECIFICATION_V3).
- `with_mock` is forbidden in production `operation` implementations (`WOLIO-EFFECT-002`) (IMPLEMENTATION_SPECIFICATION_V3).

## Acceptance criteria
Positive and boundary:
- [ ] `test` blocks calling operations that use `std::time::now`, `std::fs::read_text`, `std::fs::write_text`, and `std::net::http_send` inside a covering `with_mock { ... } in { ... }` block pass `wolio check`, `wolio test`, and `wolio verify` deterministically offline
- [ ] Nested `with_mock` blocks override inner handlers cleanly and restore outer handlers on scope exit
- [ ] Preconditions (`requires`) and postconditions (`ensures`) on mocked operations still execute and catch contract violations (`WOLIO-CONTRACT-002`/`003`)

Rejection and failure:
- [ ] A `test` block directly or transitively calling `std::time::now`, `std::fs::*`, `std::net::*`, or a `Db::*` operation without a covering `with_mock` handler fails with `WOLIO-EFFECT-002`
- [ ] Using `with_mock` inside a production `operation` implementation fails with `WOLIO-EFFECT-002`
- [ ] A mock handler with a mismatched signature fails with `WOLIO-TYPE-001`, and a mock handler attempting to perform external I/O fails with `WOLIO-EFFECT-002`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T309.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T309.md` with `V3-IO` hermetic `with_mock` and `WOLIO-EFFECT-002` rejection fixtures.

## Dependencies and open items
- Depends on: T308; blocks: T310.
- Open: none.

