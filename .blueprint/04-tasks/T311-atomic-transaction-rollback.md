`.blueprint/04-tasks/T311-atomic-transaction-rollback.md`
# T311: Implement `transaction (handle) { ... }` atomic rollback and postcondition verification

| Field | Value |
|---|---|
| Milestone | `M14` |
| Priority | `P0` |
| Depends on | T310 |
| Blocks | T316 |
| Acceptance families | V3-TX (`transaction` rollback) |
| Bounded areas | `crates/wolio-ast`, `crates/wolio-parser`, `crates/wolio-semantic` (submodules `types`, `effects`), `crates/wolio-runtime`, `crates/wolio-verify` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T311.md` |

## Objective
Implement `transaction (handle) { ... }` blocks with automatic pre-transaction
state checkpointing, atomic rollback on `Result::Err` return or runtime fault,
and error-branch `ensures` postcondition verification (Taxonomy `W10`).

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 5: Stateful resource handles and `transaction` blocks), [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md) (`W10`)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.5: Atomic `transaction` blocks)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-TX`, M14 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Grammar and AST (`crates/wolio-ast`, `crates/wolio-parser`):**
  ```text
  transaction_expr = "transaction", "(", ident, ")", "{", { let_stmt }, expr, "}" ;
  ```
  ```rust
  pub struct TransactionExpr {
      pub span: Span,
      pub handle: Spanned<String>,
      pub lets: Vec<LetStmt>,
      pub body: Box<Expr>,
  }
  ```
- **Static typing & frame rules (`crates/wolio-semantic::types`, `crates/wolio-semantic::effects`):**
  - `handle` must resolve to an in-scope `resource <Type>` parameter declared in the enclosing operation's `modifies [handle]` list (`WOLIO-RESOURCE-001` otherwise).
  - The enclosing operation must declare `effects [Db::Tx]` (or the resource's transactional effect; `WOLIO-EFFECT-001` otherwise).
  - The trailing `body` expression of `transaction (handle) { ... }` must have type `Result<T, E>` (`WOLIO-TYPE-001` otherwise), which is also the type of the `transaction` expression itself.
- **Runtime checkpoint, commit, and rollback (`crates/wolio-runtime`):**
  1. On entering `transaction (handle) { ... }`, the runtime pushes a deep copy/Copy-on-Write snapshot of `handle`'s resource state onto the handle's savepoint stack (`tx_begin(handle)`). Nested `transaction (handle)` blocks create nested savepoints on the same stack.
  2. The runtime evaluates `lets` and `body`:
     - If `body` evaluates to `Result::Ok(val)`: the runtime pops and commits the savepoint (`tx_commit(handle)`) and yields `Result::Ok(val)`.
     - If `body` evaluates to `Result::Err(err)`: the runtime atomically restores `handle`'s state to the entry savepoint (`tx_rollback(handle)`) and yields `Result::Err(err)`.
     - If `body` aborts with a runtime arithmetic fault (`WOLIO-RUNTIME-001`) or callee contract fault (`WOLIO-CONTRACT-002`..`006`), the runtime restores `handle`'s state to the entry savepoint (`tx_rollback(handle)`) before propagating the fault.
  3. Because rollback occurs inside the `transaction` expression before the enclosing operation returns `Result::Err(err)`, the enclosing operation's `ensures` postcondition (e.g., `Result::Err(_) => AccountBalance(db, from) == old(AccountBalance(db, from))`) observes the rolled-back resource state and verifies that zero partial mutations escaped.

## Scope
In scope:
- Parsing and type-checking `transaction (handle) { ... }` expressions.
- Savepoint stack (`tx_begin`, `tx_commit`, `tx_rollback`) on transactional resource adapters in `wolio-runtime`.
- Fault-injected rollback verification tests (including multi-step debit-then-failing-credit and mid-transaction arithmetic/contract faults) in `wolio-verify`.

Out of scope (owner):
- Contract property fuzzing (`property`, `wolio verify --fuzz`, T312); native LLVM lowering of `transaction` blocks (T314).

## Normative requirements
- `transaction (handle) { ... }` requires `handle` to be a `resource` parameter in `modifies [handle]` and its body to return `Result<T, E>` (CORE_V3, IMPLEMENTATION_SPECIFICATION_V3).
- Whenever a `transaction` block returns `Result::Err(...)` or faults after partially mutating `handle`, the runtime must roll `handle` back to its pre-transaction snapshot before evaluating outer `ensures` postconditions (CORE_V3, V3-TASKS, AI_BUG_TAXONOMY `W10`).
- If an AI agent mutates `handle` across multiple steps *without* wrapping them in `transaction (handle) { ... }` and an later step returns `Result::Err(...)`, the error-branch `ensures` postcondition (`Balance(db, x) == old(Balance(db, x))`) fails at runtime with `WOLIO-CONTRACT-003` (CORE_V3, AI_BUG_TAXONOMY `W10`).

## Acceptance criteria
Positive and boundary:
- [ ] A two-step `Transfer(db, from, to, amount)` inside `transaction (db) { ... }` commits both `Debit` and `Credit` on `Result::Ok` and satisfies `ensures`
- [ ] When `Debit` mutates `db` and `Credit` subsequently returns `Result::Err(TxError::AccountFrozen)` inside `transaction (db) { ... }`, `db` is rolled back to its entry snapshot and the error-branch `ensures` (`AccountBalance(db, from) == old(AccountBalance(db, from))`) passes
- [ ] Nested `transaction (db)` blocks roll back inner savepoints on inner `Err` without losing outer committed mutations

Rejection and failure:
- [ ] Multi-step mutation omitting `transaction (db)` where step 1 mutates `db` and step 2 returns `Result::Err` fails the operation's rollback `ensures` clause with `WOLIO-CONTRACT-003`
- [ ] Fault injection (arithmetic overflow or contract violation) mid-transaction rolls back `db` to its entry snapshot before aborting
- [ ] `transaction (x)` where `x` is not a `resource` in `modifies [x]` fails with `WOLIO-RESOURCE-001`, and a non-`Result` body fails with `WOLIO-TYPE-001`

Completion:
- [ ] Transaction rollback fixtures pass across interpreter and native runtime
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T311.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T311.md` with `V3-TX` fault-injected transaction rollback fixtures.

## Dependencies and open items
- Depends on: T310; blocks: T316.
- Open: none.


