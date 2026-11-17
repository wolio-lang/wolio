`.blueprint/04-tasks/T312-property-fuzz-shrinking-verifier.md`
# T312: Implement `property` declarations and `wolio verify --fuzz` shrinking counterexamples (`WOLIO-FUZZ-001`)

| Field | Value |
|---|---|
| Milestone | `M15` |
| Priority | `P0` |
| Depends on | T319 (M14 completion) |
| Blocks | T313 |
| Acceptance families | V3-FUZZ (`property`, `wolio verify --fuzz`, and `WOLIO-FUZZ-001`) |
| Bounded areas | `crates/wolio-ast`, `crates/wolio-parser`, `crates/wolio-semantic` (submodules `resolver`, `graph`, `contracts`, `analyzer`), `crates/wolio-verify`, `crates/wolio-cli` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T312.md` |

## Objective
Implement `property <name> for <op>(<params>) { where <pred>; <bool_expr> }`
declarations and the deterministic contract-driven property fuzzer and
value-tree shrinker (`wolio verify --fuzz`, fixed seed `0x574F4C494F33`,
`WOLIO-FUZZ-001`) so AI implementations that hardcode unit-test literals are
automatically defeated with minimal counterexamples (Taxonomy `W05`).

## Authority
- Normative: [CORE_V3.md](../02-specifications/language/CORE_V3.md) (Section 6: Contract-driven property fuzzing), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md), [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md) (`W05`)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md) (Section 3.6: Contract-driven property fuzzer and deterministic shrinker)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-FUZZ`, M15 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Grammar and AST (`crates/wolio-ast`, `crates/wolio-parser`):**
  ```text
  property_decl = "property", ident, "for", ident, "(", [ fields ], ")", "{", [ "where", expr, ";" ], { let_stmt }, expr, "}" ;
  ```
  ```rust
  pub struct PropertyDecl {
      pub span: Span,
      pub name: Spanned<String>,
      pub target_op: Spanned<String>,
      pub params: Vec<FieldDecl>,
      pub where_clause: Option<Box<Expr>>,
      pub lets: Vec<LetStmt>,
      pub body: Box<Expr>,
  }
  ```
- **Identity, resolution, and static rules (`crates/wolio-semantic::resolver`, `crates/wolio-semantic::graph`, `crates/wolio-semantic::contracts`, `crates/wolio-semantic::analyzer`):**
  - `property` declarations receive deterministic `sym_` IDs with `kind = "property"` under hash domain `"wolio.symbol.v1"`.
  - `target_op` must resolve to a visible `operation`, and `params` must match `target_op`'s parameter count and types (`WOLIO-TYPE-001`).
  - `where_clause` (if present) must be a pure `Bool` expression (same purity rules as `requires`: no `operation` calls, `WOLIO-CONTRACT-001`).
  - `body` must have type `Bool`, must call `target_op`, and if `target_op` has non-console effects (`Clock`, `Fs`, `Net`, `Db`), the call must be wrapped in `with_mock { ... } in { ... }` (`WOLIO-EFFECT-002`).
  - Analyzer check (`WOLIO-ANALYZER-001`): rejects any `property` whose `where` clause is literal/constant `false`, whose `body` is literal/constant `true`, or whose `body` ignores any declared parameter.
- **Deterministic Generator & Value-Tree Shrinker (`crates/wolio-verify`):**
  - Default seed: `0x574F4C494F33_u64` (`"WOLIO3"`), combined with the first 8 bytes of `target_op`'s `sym_` hash via SplitMix64.
  - Default sample count: `256` valid inputs per `property` (and per `operation` with `requires`/`ensures` when `wolio verify --fuzz` runs).
  - **Phase 1 (Boundary extraction):** Extracts comparison constants `C` from `requires` and `where` ASTs (`x >= C`, `x <= C`, `x == C`, `List::len(xs) <= C`) and tests combinations of `{C - 1, C, C + 1, 0, 1, -1, i64::MIN, i64::MAX}`.
  - **Phase 2 (PRNG synthesis):** Generates `Int`, `UInt`, `Decimal`, `Bool`, `String`, `Bytes`, nominal wrappers, records, enums, `List<T>`, and `Map<K, V>` values satisfying both `target_op`'s `requires` and the `property`'s `where` clause. If `< 16` valid inputs are found within `4096` attempts, emits `WOLIO-CONTRACT-001` (vacuously restrictive precondition).
  - **Phase 3 (Deterministic shrinking on failure):** If any generated input violates `target_op`'s `ensures` (`WOLIO-CONTRACT-003`), causes the `property` body to return `false`, or triggers an arithmetic fault (`WOLIO-RUNTIME-001` / `WOLIO-CONTRACT-004`), the shrinker binary-searches integers/decimals toward `0` (or the lower `where`/`requires` bound) and removes list/map/string elements while maintaining `requires && where == true` to find the canonical minimal counterexample.
  - Emits `WOLIO-FUZZ-001` (`severity: "error"`, `category: "fuzz"`) with `violation: { "seed": "0x574F4C494F33", "property": ..., "target_operation": ..., "counterexample": { "<param>": "<value>", ... }, "shrunk_steps": k }` and records a `property_fuzz` obligation status (`pass` or `fail`) in the `verify` evidence output.

## Scope
In scope:
- Parsing, resolving, `sym_` hashing, type-checking, and analyzer validation of `property` declarations.
- Deterministic boundary + SplitMix64 PRNG input generator (`seed = 0x574F4C494F33`, 256 cases) and minimal counterexample shrinker in `wolio-verify`.
- `wolio verify --fuzz` CLI flag (and default activation in V3 `verify` profile) emitting `WOLIO-FUZZ-001` and `property_fuzz` evidence records.

Out of scope (owner):
- Agent Protocol v3 AST payloads for `property` and mandatory `--fuzz` gate in `wolio-agent` (T313).

## Normative requirements
- `wolio verify` in V3 synthesizes deterministic boundary and PRNG inputs (default 256 cases per operation/property using fixed seed `0x574F4C494F33`) satisfying `requires`/`where` and checks both `ensures` and the `property` body (CORE_V3, V3-TASKS).
- When a generated input violates `ensures` or the `property` body, `wolio-verify` shrinks the input deterministically to a minimal counterexample and emits `WOLIO-FUZZ-001` (EVIDENCE_MODEL, V3-TASKS).
- An AI implementation that hardcodes responses for the 2–3 unit `test` literal inputs (e.g., `if (x == 10) { 10 } else if (x == 20) { 10 } else { 999 }`) must be caught deterministically by `wolio verify --fuzz` with `WOLIO-FUZZ-001` (CORE_V3, AI_BUG_TAXONOMY `W05`).

## Acceptance criteria
Positive and boundary:
- [ ] Valid `property` declarations over pure and `with_mock`-wrapped operations execute 256 deterministic generated inputs and record `method: "property_fuzz", status: "pass"`
- [ ] Re-running `wolio verify --fuzz` produces identical generated sequences and identical shrunk counterexamples across runs
- [ ] Boundary extractor automatically tests `requires`/`where` threshold edges (`lo`, `lo + 1`, `hi - 1`, `hi`)

Rejection and failure:
- [ ] A hardcoded/overfitted operation implementation that passes unit `test` blocks but fails on general inputs is caught by `wolio verify --fuzz` with `WOLIO-FUZZ-001` and a shrunk minimal counterexample (e.g., shrinking `x = 847291` down to `x = 11`)
- [ ] A vacuous `property` (`where false;` or `true` body or ignoring parameters) fails with `WOLIO-ANALYZER-001`
- [ ] Unsatisfiable `where` preconditions (`x > 10 && x < 5`) fail with `WOLIO-CONTRACT-001`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T312.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T312.md` with `V3-FUZZ` hardcoded-branch defeat fixtures and shrinking counterexample transcripts.

## Dependencies and open items
- Depends on: T319; blocks: T313.
- Open: none.

