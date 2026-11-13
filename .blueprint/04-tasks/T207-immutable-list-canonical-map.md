`.blueprint/04-tasks/T207-immutable-list-canonical-map.md`
# T207: Implement immutable `List<T>` and canonical-order `Map<K, V>`

| Field | Value |
|---|---|
| Milestone | `M09` |
| Priority | `P0` |
| Depends on | T206 |
| Blocks | T208 |
| Acceptance families | V2-DATA (collections) and V2-LOOP (collection iteration) |
| Bounded areas | `crates/wolio-resolver`, `crates/wolio-types`, `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T207.md` |

## Objective
Implement type checking and deterministic value-semantic runtime operations for
immutable `List<T>` and canonical-order `Map<K, V>`, including bounded `for`
loop iteration over `List<T>` and `Map::keys(...)`.

## Authority
- Normative: [CORE_V2.md](../02-specifications/language/CORE_V2.md) (Section 3: Builtin types and deterministic operations; Section 4: Total bounded loop semantics)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md) (Section 3.2: Value-semantic immutable `List<T>` and `Map<K, V>`)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-DATA` and `V2-LOOP`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Builtin generic types (`crates/wolio-types`):**
  - `List<T>`: accepts any valid V2 value type `T`.
    - `List::empty<T>() -> List<T>`
    - `List::push<T>(list: List<T>, item: T) -> List<T>`
    - `List::len<T>(list: List<T>) -> Int`
    - `List::get<T>(list: List<T>, index: Int) -> Option<T>` (returns `Option::None` if `index < 0` or `index >= len`; never panics or faults)
  - `Map<K, V>`: `V` may be any valid V2 value type; **`K` must be `Int`, `UInt`, `String`, `Bool`, or a `nominal` type wrapping one of those four primitives**. Any other key type (`Decimal`, `Bytes`, `Record`, `Enum`, `List`, `Map`, `Option`, `Result`, `Unit`) fails type checking with `WOLIO-TYPE-001`.
    - `Map::empty<K, V>() -> Map<K, V>`
    - `Map::insert<K, V>(map: Map<K, V>, key: K, value: V) -> Map<K, V>`
    - `Map::get<K, V>(map: Map<K, V>, key: K) -> Option<V>`
    - `Map::remove<K, V>(map: Map<K, V>, key: K) -> Map<K, V>`
    - `Map::len<K, V>(map: Map<K, V>) -> Int`
    - `Map::keys<K, V>(map: Map<K, V>) -> List<K>`
- **Canonical ordering & value equality (`crates/wolio-runtime`):**
  - `Map<K, V>` stores entries in a deterministic `BTreeMap<MapKey, Value>` ordered by `MapKey`:
    - `Bool`: `false < true`
    - `Int`: signed 64-bit ascending (`i64::MIN ..= i64::MAX`)
    - `UInt`: unsigned 64-bit ascending (`0 ..= u64::MAX`)
    - `String`: UTF-8 byte lexicographical order
    - Nominal wrapper over primitive: ordered by underlying primitive value.
  - Structural equality `==` and `!=` on `List<T>` and `Map<K, V>` are permitted when element/value types support equality.
  - Calling `List::push`, `Map::insert`, or `Map::remove` returns a new collection while leaving all existing bindings to the previous collection unchanged.

## Scope
In scope:
- Type application and builtin method checking for `List<T>` and `Map<K, V>` in `wolio-types`.
- Immutable runtime evaluation of `List` and `Map` operations and `for (item in list_expr with acc: A = init)` iteration in `wolio-runtime`.
- Usage of `List::len`, `List::get`, `Map::len`, `Map::get`, and `Map::keys` inside contract predicates (`requires`, `ensures`, `bound`, `invariant`).

Out of scope (owner):
- `format(...)` (T208); `predicate` declarations (T209); native LLVM runtime ABI for collections (T213); copy-on-write refcount optimization (V3 `T302`).

## Normative requirements
- `List<T>` and `Map<K, V>` are pure, immutable, value-semantic collections: `List::push`, `Map::insert`, and `Map::remove` never mutate existing bindings (CORE_V2).
- `Map<K, V>` keys `K` are restricted to `Int`, `UInt`, `String`, `Bool`, or nominal types over them (`WOLIO-TYPE-001` otherwise), and `Map::keys` always returns keys in deterministic canonical order independent of insertion order (CORE_V2, IMPLEMENTATION_SPECIFICATION_V2).
- `List::get(list, idx)` returns `Option::None` for negative or out-of-bounds indices and `Option::Some(elem)` for `0 <= idx < List::len(list)` (CORE_V2).
- `for (item in list_expr with acc: A = init)` evaluates `list_expr` once and iterates elements from index `0` to `List::len(list) - 1` under the `T206` `bound` and `invariant` rules (CORE_V2).

## Acceptance criteria
Positive and boundary:
- [ ] `List::empty`, `List::push`, `List::len`, and `List::get` preserve immutability of prior bindings and handle negative/out-of-bounds indices via `Option::None`
- [ ] `Map::empty`, `Map::insert`, `Map::get`, `Map::remove`, `Map::len`, and `Map::keys` work with primitive and nominal key types (`type OrderId = String;`)
- [ ] Two maps populated with the same entries in different insertion orders are `==` and yield identical `Map::keys` lists
- [ ] Bounded `for` loops iterate over `List<T>` and `Map::keys(map)` and verify `bound`/`invariant` clauses referencing `List::len` / `Map::len`

Rejection and failure:
- [ ] `Map<K, V>` with an invalid key type (`Map<Decimal, Int>`, `Map<List<Int>, Int>`, record, or enum) fails with `WOLIO-TYPE-001`
- [ ] Arity or type argument mismatches on `List::*` or `Map::*` calls fail with `WOLIO-TYPE-001`
- [ ] Using `List<T>` or `Map<K, V>` under `[language] version = "1"` fails with `WOLIO-FEATURE-001`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T207.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T207.md` with `V2-DATA` collection and `V2-LOOP` list-iteration conformance fixtures.

## Dependencies and open items
- Depends on: T206; blocks: T208.
- Open: none.

