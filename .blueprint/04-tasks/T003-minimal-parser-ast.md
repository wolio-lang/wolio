`.blueprint/04-tasks/T003-minimal-parser-ast.md`
# T003: Implement parser and AST for CORE_V1 grammar

| Field | Value |
|---|---|
| Milestone | `M01` |
| Priority | `P0` |
| Depends on | T002 |
| Blocks | T004 |
| Acceptance families | FRONT (grammar cases) |
| Bounded areas | parser and AST crates (`crates/wolio-parser`, `crates/wolio-ast`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T003.md` |

## Objective
Parse all `CORE_V1` token streams into a deterministic, spanned AST and reject
malformed declarations, chained comparisons, and reserved V2/V3 keywords used as
identifiers.

## Authority
- Normative: [CORE_V1.md](../02-specifications/language/CORE_V1.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `FRONT`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Precedence ladder (lowest to highest, left-associative for binary ops):**
  1. `||`
  2. `&&`
  3. `==`, `!=` (non-associative: `a == b == c` is rejected with `WOLIO-SYNTAX-001`)
  4. `<`, `<=`, `>`, `>=` (non-associative: `a < b < c` is rejected with `WOLIO-SYNTAX-001`)
  5. `+`, `-`
  6. `*`, `/`, `%`
  7. Unary prefix `!`, `-` (applies after postfix field selection: `-x.value` parses as `-(x.value)`)
  8. Postfix field access `.ident`
- **Integer literal range:** Positive literals must fit in `0..=9223372036854775807` (`i64::MAX`). Magnitude `9223372036854775808` is legal **only** immediately under unary `-` (producing `i64::MIN`); any other out-of-range integer literal fails with `WOLIO-SYNTAX-001`.
- **Reserved keywords as identifiers:** Any V1 keyword or V2/V3 forward-reserved keyword in an `ident` position fails with `WOLIO-SYNTAX-001`.

## Scope
In scope: every `CORE_V1` EBNF production, precedence, associativity, integer literal boundary check, and spanned AST representation.

Out of scope (owner):
- CLI and JSON envelopes (T004); name resolution and bindings (T020); typing (T021–T022); comment-preserving canonical printing (T072).

## Normative requirements
- Grammar is the `CORE_V1` EBNF, grammar version `1`; no implicit semicolon insertion, no trailing commas (CORE_V1, INTERFACE_CONTRACTS).
- Parentheses around `if (expr)` conditions and `match (expr)` subjects are mandatory to disambiguate record construction `Name { ... }` from blocks (CORE_V1).
- Syntax errors emit `WOLIO-SYNTAX-001` with exact UTF-8 byte spans.

## Acceptance criteria
- [ ] Empty module and hello-world examples parse into spanned AST nodes
- [ ] Every `CORE_V1` declaration (`type`, `record`, `enum`, `operation`, `implementation`, `test`), clause, pattern, and expression production has `FRONT` test coverage
- [ ] Precedence and `-x.value` binding order are verified by structural AST tests
- [ ] Malformed declarations, trailing commas, chained comparisons (`a < b < c`, `a == b == c`), and V1/V2/V3 reserved keywords used as identifiers are rejected with `WOLIO-SYNTAX-001`
- [ ] Integer literal boundaries (`9223372036854775807`, `-9223372036854775808` accepted; bare `9223372036854775808` rejected) are covered
- [ ] Common gates in EXECUTION_CONTRACTS pass; report written

## Verification
```bash
cargo test -p wolio-parser -p wolio-ast
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T003.md`

## Dependencies and open items
- Depends on: T002; blocks: T004.
- Open: none.
