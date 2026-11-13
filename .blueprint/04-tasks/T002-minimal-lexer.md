`.blueprint/04-tasks/T002-minimal-lexer.md`
# T002: Implement lexer for minimal source slice and V1–V3 keyword reservation

| Field | Value |
|---|---|
| Milestone | `M01` |
| Priority | `P0` |
| Depends on | T001 |
| Blocks | T003 |
| Acceptance families | FRONT (lexical cases) |
| Bounded areas | lexer crate (`crates/wolio-lexer`), shared diagnostic values |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T002.md` |

## Objective
Tokenize the complete `CORE_V1` lexical vocabulary and reserve all V1, V2, and V3
keywords with deterministic zero-based UTF-8 byte spans and structured errors.

## Authority
- Normative: [CORE_V1.md](../02-specifications/language/CORE_V1.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md) (lexical and binding clarifications), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md) (spans, codes)
- Decision: [ADR-008](../01-architecture/decisions/ADR-008-multi-version-anti-workaround-roadmap.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `FRONT`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Punctuation & operators:** `;`, `,`, `:`, `::`, `.`, `(`, `)`, `{`, `}`, `[`, `]`, `->`, `=>`, `=`, `==`, `!=`, `<`, `<=`, `>`, `>=`, `+`, `-`, `*`, `/`, `%`, `!`, `&&`, `||`.
- **V1 active keywords (22):** `module`, `import`, `pub`, `type`, `record`, `enum`, `operation`, `requires`, `ensures`, `effects`, `dependencies`, `errors`, `intent`, `implementation`, `test`, `for`, `let`, `if`, `else`, `match`, `true`, `false`.
- **V2 & V3 forward-reserved keywords (19):** `in`, `with`, `bound`, `invariant`, `predicate`, `List`, `Map`, `Decimal`, `UInt`, `Bytes`, `format`, `secret`, `Secret`, `resource`, `transaction`, `modifies`, `old`, `property`, `std`. (Tokenize as reserved keywords so parser/binder rejects their use as identifiers in V1 with `WOLIO-SYNTAX-001`.)
- **Literals:**
  - `integer`: ASCII `[0-9]+` (no `_` separators; `-` is a separate unary operator token).
  - `string`: `"..."` with JSON escapes (`\"`, `\\`, `\/`, `\b`, `\f`, `\n`, `\r`, `\t`, `\uXXXX` including valid UTF-16 surrogate pairs decoded to one Unicode scalar; unescaped `U+0000`..`U+001F` and unpaired surrogates fail with `WOLIO-SYNTAX-001`).

## Scope
In scope: the complete `CORE_V1` lexical vocabulary and V1–V3 keyword reservation in `crates/wolio-lexer`.

Out of scope (owner):
- Grammar, precedence and integer-literal range in context (T003).
- CLI output envelopes (T004); semantic validation (T020 onward).

## Normative requirements
- Source is UTF-8; invalid UTF-8 is rejected with `WOLIO-SYNTAX-001` (CORE_V1, ACCEPTANCE_MATRIX).
- Identifiers are ASCII `[A-Za-z_][A-Za-z0-9_]*`, case sensitive; all V1 active keywords and V2/V3 forward-reserved keywords are reserved in every position (CORE_V1, INTERFACE_CONTRACTS).
- `//` comments end at newline; no block comments (`/* */`). CRLF and LF are whitespace (INTERFACE_CONTRACTS).
- Spans are zero-based original UTF-8 byte offsets, `start` inclusive, `end` exclusive (EVIDENCE_MODEL, INTERFACE_CONTRACTS). All lexical errors emit `WOLIO-SYNTAX-001`.

## Acceptance criteria
- [ ] Valid minimal examples tokenize deterministically
- [ ] Invalid characters produce structured `WOLIO-SYNTAX-001` diagnostics
- [ ] All V1 active keywords and V2/V3 forward-reserved keywords are distinguished from ordinary identifiers
- [ ] Unicode strings, surrogate pairs, and CRLF inputs yield exact original UTF-8 byte spans
- [ ] Invalid UTF-8, invalid token, unescaped control character, unpaired surrogate, invalid escape, and unfinished string are rejected with `WOLIO-SYNTAX-001`
- [ ] Common gates in EXECUTION_CONTRACTS pass; report written

## Verification
```bash
cargo test -p wolio-lexer
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T002.md`

## Dependencies and open items
- Depends on: T001; blocks: T003.
- Open: none.
