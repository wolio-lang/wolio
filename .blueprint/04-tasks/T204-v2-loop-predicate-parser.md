`.blueprint/04-tasks/T204-v2-loop-predicate-parser.md`
# T204: Parse `CORE_V2` bounded `for` loops, range expressions and `predicate` declarations

| Field | Value |
|---|---|
| Milestone | `M09` |
| Priority | `P0` |
| Depends on | T203 |
| Blocks | T205, T206, T209 |
| Acceptance families | V2-LOOP (syntax cases) |
| Bounded areas | `crates/wolio-lexer`, `crates/wolio-ast`, `crates/wolio-parser` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T204.md` |

## Objective
Extend the lexer, AST, and parser to support all `CORE_V2.md` syntax extensions—
total bounded `for (item in source with acc: Type = init) { bound ...; invariant ...; ... }`
loops, `start..end` range expressions, `predicate` declarations, and `format(...)`
expressions—while rejecting them with `WOLIO-FEATURE-001` when `[language] version = "1"`.

## Authority
- Normative: [CORE_V2.md](../02-specifications/language/CORE_V2.md) (Section 2: Grammar extensions over CORE_V1), [CORE_V1.md](../02-specifications/language/CORE_V1.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-LOOP`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Lexer addition (`crates/wolio-lexer`):**
  - Add `TokenKind::DotDot` (`..`) for range expressions (`start..end`). Single `.` remains `TokenKind::Dot` (field access).
  - V2 keywords (`for`, `in`, `with`, `bound`, `invariant`, `predicate`, `List`, `Map`, `Decimal`, `UInt`, `Bytes`, `format`) and all 23 V1+V2+V3 reserved keywords (including `with_mock`, `where`, `fn`, `foreign`) are already tokenized by `T002`.
- **AST additions (`crates/wolio-ast`):**
  ```rust
  pub struct PredicateDecl {
      pub span: Span,
      pub is_pub: bool,
      pub name: Spanned<String>,
      pub params: Vec<FieldDecl>,
      pub return_type: TypeAst,
      pub body: Expr,
  }

  pub enum IterSource {
      Collection(Box<Expr>),
      Range { start: Box<Expr>, end: Box<Expr>, span: Span },
  }

  pub struct ForExpr {
      pub span: Span,
      pub item_binding: Spanned<String>,
      pub source: IterSource,
      pub acc_binding: Spanned<String>,
      pub acc_type: TypeAst,
      pub acc_init: Box<Expr>,
      pub bound_expr: Box<Expr>,
      pub invariant_expr: Box<Expr>,
      pub lets: Vec<LetStmt>,
      pub body: Box<Expr>,
  }

  // Expr::Call extended to support turbofish type arguments:
  // pub struct CallExpr {
  //     pub callee: Box<Expr>,
  //     pub turbofish: Option<Vec<TypeAst>>,
  //     pub args: Vec<Expr>,
  //     pub span: Span,
  // }
  ```
- **Grammar rules (`CORE_V2.md`):**
  - `predicate = "predicate", ident, "(", [ fields ], ")", "->", "Bool", "{", expr, "}" ;`
  - `for_expr = "for", "(", ident, "in", iter_source, "with", ident, ":", type, "=", expr, ")", "{", "bound", expr, ";", "invariant", expr, ";", { "let", ident, "=", expr, ";" }, expr, "}" ;`
  - `iter_source = expr | expr, "..", expr ;`
  - `call_expr = path, [ "::", "<", type_args, ">" ], "(", [ args ], ")" ;`
  - Omitting `bound expr;` or `invariant expr;`, or placing them in reverse order, fails parsing with `WOLIO-SYNTAX-001`.
  - Generic call expressions MUST use turbofish `path::<TArgs>(...)` when explicit type arguments are passed (e.g. `List::empty::<Int>()`), eliminating Pratt parser ambiguity with the `<` comparison operator.
  - Using `predicate`, `for`, `..`, `format(...)`, or V2 type names (`UInt`, `Decimal`, `Bytes`, `List`, `Map`) under `LanguageVersion::V1` emits `WOLIO-FEATURE-001`. Still-reserved V3 keywords (`secret`, `Secret`, `resource`, `transaction`, `modifies`, `old`, `property`, `std`, `with_mock`, `where`, `fn`, `foreign`) continue to fail with `WOLIO-SYNTAX-001` if used as identifiers in V2.

## Scope
In scope:
- `..` range token in `wolio-lexer`.
- Spanned AST nodes for `PredicateDecl`, `ForExpr`, `IterSource`, and `format(...)`, plus turbofish type argument vectors in `CallExpr` in `wolio-ast`.
- Recursive-descent/Pratt parsing of `predicate`, `for_expr`, `iter_source`, turbofish constructor calls (`List::empty::<T>()`, `Map::empty::<K, V>()`), and V2 builtin type applications in `wolio-parser`.
- Version-gate checks emitting `WOLIO-FEATURE-001` when V2 syntax is parsed under `[language] version = "1"`.

Out of scope (owner):
- Type checking and runtime evaluation of `UInt`/`Bytes`/`Decimal` (T205), `for` loops (T206), `List`/`Map` (T207), `format` (T208), and `predicate` declarations (T209).

## Normative requirements
- `for_expr` requires both `bound <expr>;` and `invariant <expr>;` in that exact order at the top of the loop block; omitting either or adding `break`/`continue`/`while` fails with `WOLIO-SYNTAX-001` (CORE_V2).
- `predicate` declaration header requires explicit `-> Bool` return type annotation (`WOLIO-SYNTAX-001` if omitted) and a single expression body `{ expr }` (CORE_V2).
- Turbofish syntax `path::<TArgs>(...)` is required for explicit generic type parameters in expression positions; parsing `name < a > (...)` as a generic call without turbofish is forbidden to prevent Pratt grammar collision with comparison operator `<` (CORE_V2).
- Under `[language] version = "1"`, V2 keywords used as identifiers fail with `WOLIO-SYNTAX-001`, and valid V2 constructs (`predicate`, `for (...)`) fail with `WOLIO-FEATURE-001` (CORE_V1, CORE_V2, ADR-008).
- V3 keywords (`secret`, `Secret`, `resource`, `transaction`, `modifies`, `old`, `property`, `std`, `with_mock`, `where`, `fn`, `foreign`) remain reserved in V2 and fail with `WOLIO-SYNTAX-001` when used as identifiers (CORE_V2, BASELINE-V2).

## Acceptance criteria
Positive and boundary:
- [ ] Valid `for (item in items with acc: Int = 0) { bound ...; invariant ...; ... }` over both collection expressions and `start..end` ranges parses into `Expr::For` with exact byte spans
- [ ] Valid `pub predicate Name(a: Int, b: Int) -> Bool { ... }` parses into `Declaration::Predicate`
- [ ] `List::empty::<Int>()`, `Map::empty::<String, Int>()`, `Decimal`, `UInt`, `Bytes`, and `format("...", ...)` parse deterministically under `LanguageVersion::V2`
- [ ] Expression `a < b` unambiguously parses as a binary comparison, not a generic call

Rejection and failure:
- [ ] `for` loop missing `bound`, missing `invariant`, missing `with acc: Type = init`, or reversing `bound`/`invariant` order fails with `WOLIO-SYNTAX-001`
- [ ] `predicate` missing `-> Bool` or attempting a `requires`/`ensures` contract block fails with `WOLIO-SYNTAX-001`
- [ ] Valid V2 `for` or `predicate` syntax in a `[language] version = "1"` project fails with `WOLIO-FEATURE-001`
- [ ] V3 reserved keywords used as identifiers in V2 fail with `WOLIO-SYNTAX-001`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T204.md`

## Verification
```bash
cargo test -p wolio-lexer -p wolio-parser
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T204.md` with V2 syntax fixture list.

## Dependencies and open items
- Depends on: T203; blocks: T205, T206, T209.
- Open: none.

