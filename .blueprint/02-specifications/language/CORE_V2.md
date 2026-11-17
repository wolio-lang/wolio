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
  Standard `*` and `/` operators trap if remainder exceeds scale `10^-6`, while
  explicit `mul_round` and `div_round` builtins require explicit, audited rounding modes.
- **Checked bitwise operations:** Systems calculations use checked functional primitives
  `bit_and`, `bit_or`, `bit_xor`, `bit_shl`, `bit_shr`, `bit_not` with strict bounds
  checks on shift amounts (`0 <= n < 64`) to eliminate undefined behavior and silent wraparounds.
- **Unambiguous generic constructor syntax:** Pratt/recursive-descent parsing uses
  turbofish syntax (`List::empty::<T>()`, `Map::empty::<K, V>()`, `Set::empty::<T>()`) or target type inference,
  completely eliminating syntactic collisions with the `<` comparison operator.
- **Canonical Content-Addressed Sets & Maps (ADR-009):** `Set<T>` and `Map<K, V>` are pure,
  immutable value collections with deterministic total ordering and canonical layouts.
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

call_expr = path, [ "::", "<", type_args, ">" ], "(", [ args ], ")" ;
type_args = type, { ",", type } ;
```

`for_expr` is an additional `primary` expression alternative in `CORE_V2`.
In expression contexts, explicit type arguments to generic functions/constructors
MUST use turbofish syntax `path::<T1, T2>(...)` to eliminate Pratt-parsing ambiguity
with the `<` comparison operator. When the target type is uniquely determined by
declaration type annotation (e.g. `let xs: List<Int> = List::empty();`), turbofish
may be omitted.

## 3. Builtin types and deterministic operations

V2 adds the following builtin types to V1's `Bool`, `Int`, `String`, `Unit`,
`Option<T>`, and `Result<T,E>`:

- `UInt`: unsigned 64-bit integer (`0 ..= 18446744073709551615`); checked arithmetic
  (`+ - * / %`); underflow below 0 or overflow above `u64::MAX` faults (`WOLIO-RUNTIME-001`).
- `RoundingMode`: builtin enum for explicit decimal rounding:
  ```wol
  enum RoundingMode {
      HalfEven,
      HalfUp,
      Floor,
      Ceil,
      Truncate
  }
  ```
- `Decimal`: exact signed 128-bit fixed-point decimal (scale 6, exact to `10^-6`);
  literals written `Decimal::from_scaled(int_mantissa, scale)` or `Decimal::of(Int)`;
  checked `+ - * / %` and ordering.
  - **Strict trapping operators:** Standard `*` and `/` operators fault (`WOLIO-RUNTIME-001`)
    if division by zero occurs or if the calculation leaves a fractional remainder
    beyond scale `10^-6` without explicit rounding (`(a * 1_000_000) % b != 0`).
  - **Explicit rounding operations:**
    - `Decimal::mul_round(a: Decimal, b: Decimal, mode: RoundingMode) -> Decimal`
    - `Decimal::div_round(a: Decimal, b: Decimal, mode: RoundingMode) -> Decimal`
    (division by zero faults with `WOLIO-RUNTIME-001`; inexact remainders are resolved
    strictly according to `mode`).
- **Checked bitwise operations (Int and UInt):**
  Functional checked primitives for systems and bitfield calculations:
  - `bit_and(a: T, b: T) -> T`
  - `bit_or(a: T, b: T) -> T`
  - `bit_xor(a: T, b: T) -> T`
  - `bit_not(a: T) -> T`
  - `bit_shl(a: T, n: Int) -> T`
  - `bit_shr(a: T, n: Int) -> T` (arithmetic shift for `Int`, logical shift for `UInt`)
  Rules: Both operands of binary bitwise operations must be of the same type (`Int` with `Int`,
  `UInt` with `UInt`). The shift amount `n` must satisfy `0 <= n < 64`. Shift amounts `< 0`
  or `>= 64` fault immediately with `WOLIO-RUNTIME-001` (or `WOLIO-CONTRACT-004` inside contracts);
  there is zero silent wraparound or undefined behavior.
- `Bytes`: immutable byte sequence. Builtins: `Bytes::empty()`, `Bytes::from_utf8(String)`,
  `Bytes::len(Bytes) -> Int`, `==`, `!=`.
- `List<T>`: immutable, value-semantic ordered sequence. Builtin constructors and
  pure total operations:
  - `List::empty::<T>() -> List<T>` (or `List::empty()` with type inference)
  - `List::push::<T>(List<T>, T) -> List<T>`
  - `List::len::<T>(List<T>) -> Int`
  - `List::get::<T>(List<T>, Int) -> Option<T>`
- `Map<K, V>`: immutable, deterministic canonical-order key-value map (`K` must be
  `Int`, `UInt`, `String`, `Bool`, or a nominal wrapper over them). Builtin pure
  operations:
  - `Map::empty::<K, V>() -> Map<K, V>` (or `Map::empty()` with type inference)
  - `Map::insert::<K, V>(Map<K, V>, K, V) -> Map<K, V>`
  - `Map::get::<K, V>(Map<K, V>, K) -> Option<V>`
  - `Map::remove::<K, V>(Map<K, V>, K) -> Map<K, V>`
  - `Map::len::<K, V>(Map<K, V>) -> Int`
  - `Map::keys::<K, V>(Map<K, V>) -> List<K>`
- `Set<T>`: immutable, deterministic canonical-order value set (`T` must be
  `Int`, `UInt`, `String`, `Bool`, or a nominal wrapper over them, matching `Map` key rules per ADR-009).
  Builtin pure operations:
  - `Set::empty::<T>() -> Set<T>` (or `Set::empty()` with type inference)
  - `Set::insert::<T>(Set<T>, T) -> Set<T>`
  - `Set::contains::<T>(Set<T>, T) -> Bool`
  - `Set::remove::<T>(Set<T>, T) -> Set<T>`
  - `Set::len::<T>(Set<T>) -> Int`
  - `Set::items::<T>(Set<T>) -> List<T>`
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

1. **Source evaluation:** `iter_source` evaluates once to a `List<T>`, `Set<T>`, or an
   integer range `start..end` (`start <= end`, exclusive of `end`; `start > end`
   yields 0 iterations). When iterating `Set<T>`, elements are traversed in ascending
   canonical order.
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
  or `[architecture]` policy). Canonical Wolio source code snippet strings (`code: string`)
  are accepted in mutation payloads and parsed server-side into verified AST diffs.
- Forward-reserved V3 keywords (`secret`, `Secret`, `resource`, `transaction`, `modifies`,
  `old`, `property`, `std`, `with_mock`, `where`, `fn`, `foreign`) remain reserved in V2
  and continue to produce `WOLIO-SYNTAX-001` when used as identifiers.

