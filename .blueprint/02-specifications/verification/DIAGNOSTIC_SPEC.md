`.blueprint/02-specifications/verification/DIAGNOSTIC_SPEC.md`
# Diagnostic Schema and Anti-Workaround Analyzer Rules

Diagnostics are a primary machine interface for AI agents, not just human compiler text.

## 1. Canonical diagnostic shape

```json
{
  "schema_version": 1,
  "code": "WOLIO-EFFECT-001",
  "severity": "error",
  "category": "effect",
  "target": "sym_<lowercase-sha256>",
  "message": "Undeclared effect",
  "violation": {
    "observed": "Console::Write",
    "declared": []
  },
  "source": {
    "file": "src/example.wol",
    "start": 10,
    "end": 20
  },
  "repair": {
    "repairable": true,
    "classes": [
      "remove_operation",
      "modify_contract",
      "split_operation"
    ],
    "forbidden": [
      "suppress",
      "ignore"
    ]
  },
  "impact": {
    "dependents": [],
    "tests": []
  }
}
```

## 2. Core invariants

- Deterministic schema (`schema_version: 1`) and field meaning.
- Stable diagnostic codes; never emit vague `"something went wrong"` messages.
- Repair `classes` identify valid structured alternatives and never include
  `"suppress"`, `"ignore"`, or automatic contract weakening.
- Diagnostics sort deterministically by `(source.file, source.start, code, target)`.

## 3. The 6-Tier Machine-Enforced Diagnostic Hierarchy

To serve industrial vibe coding where the machine enforces 100% correctness without
human code inspection, all Wolio diagnostics are partitioned into a strict 6-tier
hierarchy. Higher tiers represent foundational invariants; lower tiers represent
hyper-strict stylistic and anti-cheat enforcements. Every tier is machine-actionable:

| Tier | Category Name | Description | Default Severity | Exit Code | In-Source Suppression |
|---|---|---|---|---|---|
| **Tier 0** | `FATAL_SYNTAX` | Unparseable grammar, malformed tokens, UTF-8 violations | `fatal` | `1` | Strictly Impossible |
| **Tier 1** | `TYPE_RESOLUTION` | Type mismatches, cycle rejection, unhandled Results, illegal shadowing | `error` | `1` | Strictly Impossible |
| **Tier 2** | `BEHAVIORAL_CONTRACT` | `requires`/`ensures` violations, invariant/bound failures, fuzz counterexamples | `error` | `1` | Strictly Impossible |
| **Tier 3** | `SECURITY_CAPABILITY` | Zero-unsafe violations, unmocked test I/O, secret leaks, linear handle aliasing | `error` | `1` | Strictly Impossible |
| **Tier 4** | `ANTI_HACKING_ANALYZER` | AI reward-hacking: swallowed errors (`Ok`), dependency/effect padding, test mirroring | `error` / `warning` | `1` / `0` | Strictly Impossible (`ANALYZER-001`) |
| **Tier 5** | `HYPER_STRICT_LINT` | "Fierce Complainer": unused vars, dead branches, tautological contracts, naked literals | `error` (default) | `1` | Strictly Impossible |

## 4. Hyper-Strict Linter Rules (`WOLIO-LINT-001`..`006`)

Unlike Rust's Clippy where lints are optional or can be suppressed with `#[allow(...)]`,
**Wolio explicitly treats in-source suppression directives as fatal compilation errors (`WOLIO-ANALYZER-001`)**.
The AI agent must structurally repair the code to satisfy the linter:

| Code | Severity | Meaning | Deterministic Trigger |
|---|---|---|---|
| `WOLIO-LINT-001` | `error` | Unused binding, parameter, or import | An imported symbol, operation parameter, or `let` binding is never referenced within its declaring scope. |
| `WOLIO-LINT-002` | `error` | Dead code or redundant match arm | An AST expression, branch continuation, or `match` arm is provably unreachable given prior conditions or pattern coverage. |
| `WOLIO-LINT-003` | `error` | Tautological contract clause | A `requires`, `ensures`, `bound`, or `invariant` predicate is statically identical to `true`, evaluates to a constant identity (e.g., `x == x`), or establishes no constraint on visible state. |
| `WOLIO-LINT-004` | `error` | Naked magic literal in business logic | An integer or string literal appears inside a production business calculation outside of a nominal type constructor (`CustomerId(1)`) or standard mathematical constant (`0`, `1`). |
| `WOLIO-LINT-005` | `error` | Missing intent clause | A public `operation` declaration omits an explicit `intent "..."` clause explaining its semantic purpose. |
| `WOLIO-LINT-006` | `error` | Non-Unit statement expression discard | A statement expression (`expr;`) evaluates to a type other than `Unit`, discarding a value or `Result` without explicit binding. |

## 5. V1 Anti-Workaround Analyzer Rules (`WOLIO-ANALYZER-001`..`005`)

To prevent AI coding agents from introducing reward-hacking workarounds
("akal-akalan") that pass basic typechecks while creating technical debt, `M04`
(`T044`) runs the analyzer pass after static checks and before evidence
aggregation. All five codes have concrete, deterministic triggers:

| Code | Severity | Meaning | Deterministic Trigger |
|---|---|---|---|
| `WOLIO-ANALYZER-001` | `error` | Semantic suppression or verification bypass | Any mutation request, attribute, comment directive, or repair attempt requesting diagnostic suppression, rule disabling, or verification bypass. |
| `WOLIO-ANALYZER-002` | `error` | Swallowed error substituted with constant `Ok` | Inside an implementation of an operation returning `Result<T, E>`, a `match` over a callee `Result` has a `Result::Err(_)` arm that never references its bound error variable and returns `Result::Ok(<constant_or_pure_literal_expr>)`. |
| `WOLIO-ANALYZER-003` | `error` | Declared dependency or effect bloat (padding) | An operation declares a symbol in `dependencies [...]` that is never called in its `implementation`, or declares an effect in `effects [...]` that is neither directly nor transitively produced by its called operations. (Prevents AI from blindly padding `dependencies`/`effects` lists to silence errors.) |
| `WOLIO-ANALYZER-004` | `error` | Forbidden hardcoded business constant | Production `implementation` AST contains a literal or **constant-folded pure expression** (e.g., `99 + 1` evaluating to `100`) matching an entry in `wolio.toml` `[analysis]` `forbidden_int_literals = [...]` or `forbidden_string_literals = [...]`. Tests, contract predicates, and `intent` strings are excluded. |
| `WOLIO-ANALYZER-005` | `warning` | Suspicious duplicate arms or test-literal mirroring | Emitted once per construct when either: (a) a `match` expression's normalized arm bodies are structurally identical after bound-variable alpha-renaming, OR (b) a production `if`/`match` condition compares a parameter against an exact non-boundary literal (`!= -1, 0, 1`) that appears as an input argument in a `test` block for that operation (detecting AI test-overfitting). |

### Enforcement boundary

- `WOLIO-ANALYZER-001`..`004` are `error` diagnostics: they fail `verify`, block
  `apply` and `finalize`, and require a genuine structural repair.
- `WOLIO-ANALYZER-005` is a `warning` diagnostic: it is recorded in `diagnostics`
  for human/audit visibility without failing `verify` or escalating the approval
  tier (`APPROVAL_MATRIX_V1.md`).
- Neither `WOLIO-ANALYZER-002` nor `003` duplicates `WOLIO-ERROR-001`,
  `WOLIO-DEPENDENCY-001`, or `WOLIO-EFFECT-001` (`002`/`003` run only after `T031`
  checks pass).

## 6. Scalar Boundary and Fuzz Verification Diagnostics (M03 / T032)

To defeat AI test-overfitting and reward-hacking directly at V1 M03 without
waiting for V3 arbitrary generators, `wolio verify --fuzz` tests contract-guarded
operations against automated boundary and scalar inputs:

| Code | Severity | Meaning | Deterministic Trigger |
|---|---|---|---|
| `WOLIO-VERIFY-002` | `error` | Contract postcondition violated under scalar boundary or fuzz input | Under `wolio verify --fuzz`, an operation with `requires`/`ensures` produces an `ensures` postcondition violation or runtime fault when executed with boundary scalar inputs (`Int::MIN`, `Int::MAX`, `-1`, `0`, `1`, inequality boundaries) or deterministic seeded pseudo-random inputs that satisfy `requires`. |

## 7. V3 Capability, Security, Resource, and FFI Diagnostics (`M13`–`M15`)

V3 registers strict compile-time and runtime diagnostics enforcing the capability model, linear resources, and Foreign Function Interface (FFI):

| Code | Severity | Meaning | Deterministic Trigger |
|---|---|---|---|
| `WOLIO-SECURITY-001` | `error` | Opaque `Secret<T>` violation or unvalidated provenance | Constructing `Secret<T>` from literal/pure expression in production code; attempting to access `.value`, apply `==`/`!=`, or pass `Secret<T>` to `format`, `print`, or `std::json`; or constructing domain type from `UntrustedOrigin` without validating smart-constructor `ensures`. |
| `WOLIO-EFFECT-002` | `error` | Unmocked external I/O or foreign call in test sandbox | A `test` or `property` block invokes an operation with non-console external effects (`Clock::Read`, `Fs::*`, `Net::*`, `Db::*`, `Foreign::Call`) without an enclosing `with_mock` handler covering that operation. |
| `WOLIO-RESOURCE-001` | `error` | Linear `resource` handle aliasing, leaking, or missing `modifies` | Passing the same `resource` binding twice in one call (`f(db, db)`); aliasing via `let b2 = b;`; returning a `resource` without linear transfer; embedding a `resource` in record/enum/collection; or mutating a `resource` without declaring `modifies [handle];`. |
| `WOLIO-FOREIGN-001` | `error` | Undeclared foreign library in manifest | A `foreign type` or `foreign operation` references a library in `from "<library>"` that is not listed in `wolio.toml` `[foreign].allowed_libraries`. |
| `WOLIO-FOREIGN-002` | `error` | Illegal storage of linear opaque `foreign type` | Attempting to declare a field of a `record`, an `enum` variant payload, or a collection (`List`, `Map`, `Option`, `Result`) containing a `foreign type`. |
| `WOLIO-FUZZ-001` | `error` | Contract or property fuzzing counterexample | `wolio verify --fuzz` discovers an input satisfying `requires`/`where` that causes an `ensures` violation, property body `false`, or unhandled runtime fault, returning a deterministically shrunk minimal counterexample. |


