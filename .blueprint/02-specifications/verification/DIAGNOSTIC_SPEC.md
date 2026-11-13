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

## 3. V1 Anti-Workaround Analyzer Rules (`WOLIO-ANALYZER-001`..`005`)

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
