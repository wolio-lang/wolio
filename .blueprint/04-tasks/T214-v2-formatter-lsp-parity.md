`.blueprint/04-tasks/T214-v2-formatter-lsp-parity.md`
# T214: Update `wolio fmt` and `wolio lsp` for all `CORE_V2` constructs

| Field | Value |
|---|---|
| Milestone | `M11` |
| Priority | `P0` |
| Depends on | T213 |
| Blocks | T215 |
| Acceptance families | V2-NATIVE, TOOL |
| Bounded areas | `crates/wolio-format`, `crates/wolio-lsp` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T214.md` |

## Objective
Update the canonical source formatter (`wolio fmt`) and Language Server Protocol
server (`wolio lsp`) to support all `CORE_V2` declarations, expressions, and
types with comment preservation, `sym_` ID invariance, and incremental editor
navigation.

## Authority
- Normative: [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) (Section 4: Formatter and LSP), [CORE_V2.md](../02-specifications/language/CORE_V2.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-NATIVE` and `TOOL`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Canonical V2 formatting rules (`crates/wolio-format`):**
  - `predicate` declaration:
    ```wol
    pub predicate ValidRange(lo: Int, hi: Int) -> Bool {
        lo >= 0 && lo <= hi
    }
    ```
  - Bounded `for` expression (4-space indentation, LF line endings, `bound` then `invariant` on dedicated lines):
    ```wol
    let total = for (item in items with acc: Int = 0) {
        bound List::len(items) <= 1000;
        invariant acc >= 0;
        let next = acc + item;
        next
    };
    ```
  - Preserve `//` comments attached to `predicate`, `bound`, `invariant`, and loop body statements; verify that formatting preserves every `sym_` declaration ID (including `kind = "predicate"`) and normalized contract/loop clause hash.
  - Format turbofish constructor calls canonically without spaces inside `::<...>` (e.g. `List::empty::<Int>()`, `Map::empty::<String, Int>()`).
- **LSP V2 capabilities (`crates/wolio-lsp`):**
  - Diagnostics: publish real-time `WOLIO-CONTRACT-005`/`006` static checks, `WOLIO-ARCH-001` import diagnostics, and V2 analyzer diagnostics on unsaved in-memory document overlays.
  - Hover: display `predicate` signatures, `List<T>` / `Map<K, V>` / `Decimal` / `UInt` / `Bytes` inferred types, bitwise functions, and loop `item`/`acc` binding types.
  - Go-to-definition & Find-references: resolve `predicate` calls inside `requires`, `ensures`, `bound`, `invariant`, and other `predicate`s, as well as local references to loop `item` and `acc`.
  - Workspace rename (`textDocument/rename`): safely rename `predicate` declarations (updating all callers across modules and contracts) and loop `item`/`acc` variables with document version checking.

## Scope
In scope:
- Canonical formatting of `predicate`, `for..in..with..bound..invariant`, `start..end`, `format(...)`, and V2 type applications in `wolio-format`.
- Hover, go-to-definition, find-references, diagnostics, and version-checked rename for V2 constructs in `wolio-lsp`.

Out of scope (owner):
- 24-task V2 benchmark execution (T215); V2 release packaging (T216).

## Normative requirements
- `wolio fmt` is idempotent (`fmt(fmt(x)) == fmt(x)`), preserves all `//` comments, uses 4-space indentation and LF newlines, and verifies that all `sym_` declaration IDs and clause hashes are unchanged after formatting (DELIVERY_POLICY).
- `wolio lsp` operates over the shared `wolio-semantic` model with in-memory unsaved-buffer overlays (never writing unsaved editor buffers to disk) and rejects stale-version rename edits (DELIVERY_POLICY, ACCEPTANCE_MATRIX).

## Acceptance criteria
Positive and boundary:
- [ ] `wolio fmt` formats all `CORE_V2` fixtures canonically, preserves `//` comments, passes `wolio fmt --check` on the second run, and preserves all `sym_` IDs
- [ ] `wolio lsp` returns accurate hover types, definitions, and references for `predicate` symbols, loop `item`/`acc` bindings, and `List`/`Map`/`Decimal` expressions
- [ ] Workspace rename of a `pub predicate` updates all module imports and `requires`/`ensures`/`bound`/`invariant` call sites atomically

Rejection and failure:
- [ ] `wolio fmt --check` exits non-zero (`1`) on unformatted V2 source without modifying files on disk
- [ ] `wolio lsp` rejects rename requests on stale document versions or collisions with reserved V1/V2/V3 keywords

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T214.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T214.md` with V2 formatter idempotence and LSP protocol session transcripts.

## Dependencies and open items
- Depends on: T213; blocks: T215.
- Open: none.

