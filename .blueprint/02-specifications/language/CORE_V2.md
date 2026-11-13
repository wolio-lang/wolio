`.blueprint/02-specifications/language/CORE_V2.md`
# Core language V2 — Bounded iteration, collections, pure predicates, and architecture rules

Normative for the V2 profile in `BASELINE-V2.md`. Grammar version `2` (`[language] version = "2"`).
`CORE_V2` is a strict, upward-compatible superset of `CORE_V1.md`: every valid V1
program compiles and executes identically under V2 when `[language] version = "1"`
or `"2"`. Using V2 constructs in a project declaring `[language] version = "1"`
produces `WOLIO-FEATURE-001`.

## 1. Anti-workaround design rationale for V2

Conventional languages give AI unrestricted `while (cond)` loops, mutable variables,
floating-point numbers, and implicit string concatenation—which AI routinely uses
to write infinite loops, off-by-one bugs, IEEE-754 rounding defects, and injection
vulnerabilities. V2 adds core expressiveness while keeping AI workarounds impossible:

- **No `while (true)` or unbounded recursion:** Every loop in V2 is a **total,
  bounded accumulator loop** (`for item in collection with acc = init { bound ...; invariant ...; body }`)
  that is guaranteed to terminate and checks its loop `invariant` at entry, every
  iteration step, and exit.
- **No mutable local variables:** Loop state advances functionally by returning the
  next accumulator value at the end of each iteration body.
- **No binary floating-point:** Financial and fractional domain math uses exact
  fixed-scale `Decimal` (`128-bit` scaled integer arithmetic) with checked overflow.
- **No reusable-contract duplication:** Pure, total, effect-free `predicate`
  declarations can be called inside `requires`, `ensures`, and loop `invariant`
  clauses without allowing arbitrary side-effecting operations in contracts.

## 2. Grammar extensions over CORE_V1

```ebnf
declaration = [ "pub" ], ( nominal | record | enum | predicate | operation )
            | implementation | test ;

predicate = "predicate", ident, "(", [ fields ], ")", "->", "Bool",
            "{", expr, "}" ;

for_expr = "for", "(", ident, "in", iter_source,
           "with", ident, ":", type, "=", expr, ")", "{",
           "bound", expr, ";",
           "invariant", expr, ";",
           { "let", ident, "=", expr, ";" },
           expr,
           "}" ;

iter_source = expr | expr, "..", expr ;
```

`for_expr` is an additional `primary` expression alternative in `CORE_V2`.

## 3. Builtin types and deterministic operations

V2 adds the following builtin types to V1's `Bool`, `Int`, `String`, `Unit`,
`Option<T>`, and `Result<T,E>`:

- `UInt`: unsigned 64-bit integer; checked overflow/underflow (`WOLIO-RUNTIME-001`).
- `Decimal`: exact signed 128-bit fixed-point decimal (scale 6, exact to `10^-6`);
  literals written `Decimal::from_scaled(int_mantissa, scale)` or `Decimal::of(Int)`;
  checked `+ - * / %` and ordering; never wraps or silently rounds inexact division.
- `Bytes`: immutable byte sequence.
- `List<T>`: immutable, value-semantic ordered sequence. Builtin constructors and
  pure total operations:
  - `List::empty<T>() -> List<T>`
  - `List::push<T>(List<T>, T) -> List<T>`
  - `List::len<T>(List<T>) -> Int`
  - `List::get<T>(List<T>, Int) -> Option<T>`
- `Map<K, V>`: immutable, deterministic canonical-order key-value map (`K` must be
  `Int`, `UInt`, `String`, `Bool`, or a nominal wrapper over them). Builtin pure
  operations:
  - `Map::empty<K, V>() -> Map<K, V>`
  - `Map::insert<K, V>(Map<K, V>, K, V) -> Map<K, V>`
  - `Map::get<K, V>(Map<K, V>, K) -> Option<V>`
  - `Map::remove<K, V>(Map<K, V>, K) -> Map<K, V>`
  - `Map::len<K, V>(Map<K, V>) -> Int`
  - `Map::keys<K, V>(Map<K, V>) -> List<K>`
- `format(template: String, args...) -> String`: deterministic, type-checked
  string formatting (`format("order={} total={}", id.value, total)`). Placeholders
  `{}` must match argument count and only accept `Int`, `UInt`, `Decimal`, `Bool`,
  or `String` (never raw nominal types without `.value` or `Secret<T>`).

## 4. Total bounded loop semantics (`for ... in ... with ...`)

```wol
let total = for (item in items with acc: Int = 0) {
    bound List::len(items) <= 1000;
    invariant acc >= 0;
    acc + item
};
```

1. **Source evaluation:** `iter_source` evaluates once to either a `List<T>` or an
   integer range `start..end` (`start <= end`, exclusive of `end`; `start > end`
   yields 0 iterations).
2. **Mandatory `bound` check:** Before iteration begins, `bound expr;` (type `Bool`)
   is evaluated once. For `start..end` ranges, the compiler also requires an
   explicit upper-bound comparison in `bound`. If `bound` evaluates to `false`, execution
   aborts with `WOLIO-CONTRACT-005` (`loop bound violated`).
3. **Mandatory `invariant` check:** `invariant expr;` (type `Bool`) may reference
   outer bindings and the accumulator `acc`. It is checked:
   - **On entry** (with `acc = init` before iteration 0),
   - **After each iteration step** (with `acc = next_acc`),
   - **On exit** (before yielding the final `acc` as the `for` expression value).
   Any `false` evaluation aborts with `WOLIO-CONTRACT-006` (`loop invariant violated`).
4. **No mutation or early jump:** Neither `item` nor `acc` may shadow outer bindings
   or be reassigned; there is no `break` or `continue`.

## 5. Pure `predicate` declarations

```wol
pub predicate ValidDiscountBasis(amount: Int, cap: Int) -> Bool {
    amount >= 0 && cap >= 0 && amount <= cap
}
```

- A `predicate` has no separate `implementation` block, no `effects`, and may call
  only other `predicate` declarations and builtin pure constructors/accessors.
- Predicate call graphs must be strictly acyclic (`WOLIO-RESOLVE-001` on cycles).
- `requires`, `ensures`, `bound`, and `invariant` clauses in V2 may call visible
  `predicate` declarations; calling an `operation` inside a contract remains
  forbidden (`WOLIO-CONTRACT-001`).
- `sym_` identity kind for a predicate is `"predicate"`.

## 6. Architecture boundary policy and V2 agent mutations

- `wolio.toml` in V2 supports `[architecture]` module dependency rules:
  `allow_imports = [{ from = "app", to = "domain" }]` and
  `deny_imports = [{ from = "domain", to = "infra" }]` (deny overrides allow).
  Violations fail static checking with `WOLIO-ARCH-001`.
- Protocol version `2` adds `create_module`, `delete_module`, `add_import`, and
  `remove_import` to `MUTATION_SCHEMA` (all classified `POST_REVIEW` when private/
  allowed by `[architecture]`, or `HUMAN_REQUIRED` when touching public exports
  or `[architecture]` policy).

