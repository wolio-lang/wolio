`.blueprint/04-tasks/T207-immutable-list-canonical-map.md`
# T207: Implement immutable `List<T>`, canonical-order `Map<K, V>`, and canonical `Set<T>`

| Field | Value |
|---|---|
| Milestone | `M09` |
| Priority | `P0` |
| Depends on | T206 |
| Blocks | T208 |
| Acceptance families | V2-DATA (collections) and V2-LOOP (collection iteration) |
| Bounded areas | `crates/wolio-semantic` (submodules `resolver`, `types`), `crates/wolio-runtime` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T207.md` |

## Objective
Implement type checking and deterministic value-semantic runtime operations for
immutable `List<T>`, canonical-order `Map<K, V>`, and canonical `Set<T>`, including
turbofish constructor syntax (`List::empty::<T>()`, `Map::empty::<K, V>()`, `Set::empty::<T>()`)
and bounded `for` loop iteration over `List<T>`, `Set<T>`, and `Map::keys(...)`.

## Authority
- Normative: [CORE_V2.md](../02-specifications/language/CORE_V2.md) (Section 3: Builtin types and deterministic operations; Section 4: Total bounded loop semantics)
- Decision: [ADR-009](../01-architecture/decisions/ADR-009-unconventional-paradigm-unification.md) (Pillar 2: Canonical Relational-Algebraic Value Collections)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md) (Section 3.3: Value-semantic immutable collections)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-DATA` and `V2-LOOP`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Builtin generic types (`crates/wolio-semantic::types`):**
  - `List<T>`: accepts any valid V2 value type `T`.
    - `List::empty::<T>() -> List<T>` (or `List::empty()` with type inference)
    - `List::push::<T>(list: List<T>, item: T) -> List<T>`
    - `List::len::<T>(list: List<T>) -> Int`
    - `List::get::<T>(list: List<T>, index: Int) -> Option<T>` (returns `Option::None` if `index < 0` or `index >= len`; never panics or faults)
  - `Map<K, V>`: `V` may be any valid V2 value type; **`K` must be `Int`, `UInt`, `String`, `Bool`, or a `nominal` type wrapping one of those four primitives**. Any other key type (`Decimal`, `Bytes`, `Record`, `Enum`, `List`, `Map`, `Set`, `Option`, `Result`, `Unit`) fails type checking with `WOLIO-TYPE-001`.
    - `Map::empty::<K, V>() -> Map<K, V>` (or `Map::empty()` with type inference)
    - `Map::insert::<K, V>(map: Map<K, V>, key: K, value: V) -> Map<K, V>`
    - `Map::get::<K, V>(map: Map<K, V>, key: K) -> Option<V>`
    - `Map::remove::<K, V>(map: Map<K, V>, key: K) -> Map<K, V>`
    - `Map::len::<K, V>(map: Map<K, V>) -> Int`
    - `Map::keys::<K, V>(map: Map<K, V>) -> List<K>`
  - `Set<T>`: **`T` must be `Int`, `UInt`, `String`, `Bool`, or a `nominal` type wrapping one of those four primitives** (matching `Map` key rules). Any other element type fails type checking with `WOLIO-TYPE-001`.
    - `Set::empty::<T>() -> Set<T>` (or `Set::empty()` with type inference)
    - `Set::insert::<T>(set: Set<T>, item: T) -> Set<T>`
    - `Set::contains::<T>(set: Set<T>, item: T) -> Bool`
    - `Set::remove::<T>(set: Set<T>, item: T) -> Set<T>`
    - `Set::len::<T>(set: Set<T>) -> Int`
    - `Set::items::<T>(set: Set<T>) -> List<T>` (returns items in ascending canonical order)
  - Explicit generic calls in expressions require turbofish `::<...>` syntax, preventing syntactic collision with the `<` comparison operator.
- **Canonical ordering & value equality (`crates/wolio-runtime`):**
  - `Map<K, V>` stores entries in a deterministic `BTreeMap<MapKey, Value>` ordered by `MapKey`:
    - `Bool`: `false < true`
    - `Int`: signed 64-bit ascending (`i64::MIN ..= i64::MAX`)
    - `UInt`: unsigned 64-bit ascending (`0 ..= u64::MAX`)
    - `String`: UTF-8 byte lexicographical order
    - Nominal wrapper over primitive: ordered by underlying primitive value.
  - `Set<T>` stores entries in a deterministic `BTreeSet<MapKey>` ordered identically by `MapKey`.
  - Structural equality `==` and `!=` on `List<T>`, `Map<K, V>`, and `Set<T>` are permitted when element/value types support equality.
  - Calling `List::push`, `Map::insert`, `Map::remove`, `Set::insert`, or `Set::remove` returns a new collection while leaving all existing bindings to the previous collection unchanged.

## Scope
In scope:
- Type application and builtin method checking for `List<T>`, `Map<K, V>`, and `Set<T>` in `wolio-types`.
- Immutable runtime evaluation of `List`, `Map`, and `Set` operations and `for (item in iter_expr with acc: A = init)` iteration in `wolio-runtime`.
- Usage of `List::len`, `List::get`, `Map::len`, `Map::get`, `Map::keys`, `Set::len`, `Set::contains`, and `Set::items` inside contract predicates (`requires`, `ensures`, `bound`, `invariant`).

Out of scope (owner):
- `format(...)` (T208); `predicate` declarations (T209); native LLVM runtime ABI for collections (T213); copy-on-write refcount optimization (V3 `T302`).

## Normative requirements
- `List<T>`, `Map<K, V>`, and `Set<T>` are pure, immutable, value-semantic collections: `List::push`, `Map::insert`, `Map::remove`, `Set::insert`, and `Set::remove` never mutate existing bindings (CORE_V2, ADR-009).
- `Map<K, V>` keys `K` and `Set<T>` elements `T` are restricted to `Int`, `UInt`, `String`, `Bool`, or nominal types over them (`WOLIO-TYPE-001` otherwise), and `Map::keys` / `Set::items` always returns items in deterministic canonical order independent of insertion order (CORE_V2, IMPLEMENTATION_SPECIFICATION_V2, ADR-009).
- `List::get(list, idx)` returns `Option::None` for negative or out-of-bounds indices and `Option::Some(elem)` for `0 <= idx < List::len(list)` (CORE_V2).
- `for (item in iter_expr with acc: A = init)` evaluates `iter_expr` once and iterates elements under the `T206` `bound` and `invariant` rules (CORE_V2).

## Acceptance criteria
Positive and boundary:
- [ ] `List::empty`, `List::push`, `List::len`, and `List::get` preserve immutability of prior bindings and handle negative/out-of-bounds indices via `Option::None`
- [ ] `Map::empty`, `Map::insert`, `Map::get`, `Map::remove`, `Map::len`, and `Map::keys` work with primitive and nominal key types (`type OrderId = String;`)
- [ ] `Set::empty`, `Set::insert`, `Set::contains`, `Set::remove`, `Set::len`, and `Set::items` work with primitive and nominal element types
- [ ] Two maps/sets populated with the same entries in different insertion orders are `==` and yield identical `Map::keys` / `Set::items` lists
- [ ] Bounded `for` loops iterate over `List<T>`, `Set<T>`, and `Map::keys(map)` and verify `bound`/`invariant` clauses referencing `List::len` / `Map::len` / `Set::len`

Rejection and failure:
- [ ] `Map<K, V>` or `Set<T>` with an invalid key/element type (`Map<Decimal, Int>`, `Set<Decimal>`, `Set<List<Int>>`, record, or enum) fails with `WOLIO-TYPE-001`
- [ ] Arity or type argument mismatches on `List::*`, `Map::*`, or `Set::*` calls fail with `WOLIO-TYPE-001`
- [ ] Using `List<T>`, `Map<K, V>`, or `Set<T>` under `[language] version = "1"` fails with `WOLIO-FEATURE-001`

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

