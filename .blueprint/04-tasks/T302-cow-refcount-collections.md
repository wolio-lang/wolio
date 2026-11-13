`.blueprint/04-tasks/T302-cow-refcount-collections.md`
# T302: Implement copy-on-write (CoW) reference-counting optimization for `List<T>` and `Map<K, V>`

| Field | Value |
|---|---|
| Milestone | `M12` |
| Priority | `P0` |
| Depends on | T301 |
| Blocks | T303 |
| Acceptance families | V3-REGRESSION |
| Bounded areas | `crates/wolio-runtime`, `crates/wolio-ir`, `crates/wolio-codegen` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T302.md` |

## Objective
Optimize `List::push` and `Map::insert`/`Map::remove` in `crates/wolio-runtime`,
`crates/wolio-ir`, and `crates/wolio-codegen` to mutate collection storage in
place when the runtime reference count is uniquely owned (`rc == 1`) while
preserving pure immutable value semantics whenever a collection is shared
(`rc > 1`), verified by zero-leak counters and differential parity tests.

## Authority
- Normative: [BASELINE-V3.md](../BASELINE-V3.md) (Section 1 & Section 3: M12), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md) (Section 2: Reference counting & CoW optimization), [CORE_V2.md](../02-specifications/language/CORE_V2.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.1: Copy-on-Write reference-counting optimization)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-REGRESSION`, M12 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Interpreter Copy-on-Write (`crates/wolio-runtime`):**
  - `Value::List(Arc<Vec<Value>>)` and `Value::Map(Arc<BTreeMap<MapKey, Value>>)` use `Arc::make_mut` during `List::push`, `Map::insert`, and `Map::remove`.
  - In `for (item in src with acc: List<T> = init)` and last-use local variable consumption (`let xs2 = List::push(xs1, v);` where `xs1` is not referenced later in scope), the evaluator moves `acc` / `xs1` out of the frame binding before calling `Arc::make_mut`, ensuring `Arc::strong_count == 1` and mutating the `Vec` or `BTreeMap` in place without reallocating or cloning existing elements.
  - When `xs1` is still live later in scope (`Arc::strong_count > 1`), `Arc::make_mut` clones the container before applying the update so `xs1` remains completely unchanged.
- **Native IR & C Runtime ABI (`crates/wolio-ir`, `crates/wolio-codegen`, `crates/wolio-runtime`):**
  - Heap structures `WolioListHeader { rc: AtomicU64, len: u64, cap: u64, data: *mut RawValue }` and `WolioMapHeader { rc: AtomicU64, tree: BTreeMap<MapKey, Value> }`.
  - C ABI functions:
    - `wolio_rt_list_push_cow(list: *mut WolioListHeader, elem: RawValue) -> *mut WolioListHeader`
    - `wolio_rt_map_insert_cow(map: *mut WolioMapHeader, key: RawMapKey, val: RawValue) -> *mut WolioMapHeader`
    - `wolio_rt_map_remove_cow(map: *mut WolioMapHeader, key: RawMapKey) -> *mut WolioMapHeader`
    - `wolio_rt_collection_retain(ptr: *mut c_void)` / `wolio_rt_collection_release(ptr: *mut c_void)`
    - `wolio_rt_live_allocations() -> u64` (returns active heap collection/string/bytes count for zero-leak verification).
  - In `wolio-ir`, last-use analysis marks unique collection arguments so `wolio-codegen` passes ownership directly to `*_cow` without emitting a redundant retain/release pair around loop accumulator updates.

## Scope
In scope:
- Last-use move optimization and `Arc::make_mut` in-place mutation in `wolio-runtime`.
- Last-use ownership hints in `wolio-ir` and `*_cow` C ABI lowering in `wolio-codegen`.
- Deterministic allocation/deallocation tracking (`wolio_rt_live_allocations() == 0` at exit) and CoW latency/allocation benchmarks proving linear `O(N)` total element copies (instead of quadratic `O(N^2)`) when pushing `N = 10_000` elements in a bounded `for` loop.

Out of scope (owner):
- `[language] version = "3"` manifest gating and LSP profiling (T303).
- `std::collections` helper operations (`sort_int`, `reverse`, etc. in T305).

## Normative requirements
- `List<T>` and `Map<K, V>` must preserve 100% pure value semantics: no observable mutation of any aliased variable (`rc > 1`) may ever occur in interpreter or native execution (CORE_V2, DELIVERY_POLICY, IMPLEMENTATION_SPECIFICATION_V3).
- No pointer identity or reference count value may be exposed to Wolio source programs (DELIVERY_POLICY).
- Both interpreter and native execution must complete with zero leaked heap allocations across normal returns, `Result::Err` paths, and loop iterations (BASELINE-V3, IMPLEMENTATION_SPECIFICATION_V3).

## Acceptance criteria
Positive and boundary:
- [ ] Building a `List<Int>` or `Map<Int, Int>` of `10,000` elements inside a bounded `for` loop executes in-place (`rc == 1`) with `O(N)` allocations instead of `O(N^2)` clones in both interpreter and native builds
- [ ] Branching aliases (`let b = List::push(a, 1); let c = List::push(a, 2);`) preserve `a`, `b`, and `c` as distinct immutable values with identical results in interpreter and native execution
- [ ] Nested collections (`List<List<Int>>`, `Map<String, List<Decimal>>`) maintain exact value semantics and zero memory leaks (`wolio_rt_live_allocations() == 0` on exit)

Rejection and failure:
- [ ] Any memory leak (`live_allocations != 0` after scope cleanup) or alias corruption fails the `V3-REGRESSION` test suite
- [ ] Any IR mutation instruction violating `VerifiedWolioIr` type/ownership invariants is rejected by `wolio_ir::verify_ir`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T302.md`

## Verification
```bash
cargo test -p wolio-runtime -p wolio-ir -p wolio-codegen
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T302.md` with CoW copy-count measurements (`10,000`-element loop), alias-preservation tests, and zero-leak check results.

## Dependencies and open items
- Depends on: T301; blocks: T303.
- Open: none.

