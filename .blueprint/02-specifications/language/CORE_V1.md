`.blueprint/02-specifications/language/CORE_V1.md`
# Core language V1

Normative for the V1 profiles in `BASELINE-V1.md`. Grammar version `1` (`[language] version = "1"`).
All programs are UTF-8. Identifiers are ASCII `[A-Za-z_][A-Za-z0-9_]*`, case
sensitive; keywords are reserved in every position. Whitespace is insignificant,
`//` comments end at newline. Strings use JSON escapes. Integer literals are
decimal digits with no separators; unary minus is an operator. No implicit
semicolon insertion.

## Reserved words and forward compatibility

To guarantee zero source collisions across V1, V2 (`CORE_V2.md`), and V3
(`CORE_V3.md`), the V1 lexer and binder reserve all three sets below:

1. **V1 active keywords (22):** `module`, `import`, `pub`, `type`, `record`,
   `enum`, `operation`, `requires`, `ensures`, `effects`, `dependencies`,
   `errors`, `intent`, `implementation`, `test`, `for`, `let`, `if`, `else`,
   `match`, `true`, `false`.
2. **V1 reserved builtins & bindings (13):** `Bool`, `Int`, `String`, `Unit`,
   `Option`, `Result`, `None`, `Some`, `Ok`, `Err`, `unit`, `console`, `result`.
3. **V2 & V3 forward-reserved keywords (19):** `in`, `with`, `bound`,
   `invariant`, `predicate`, `List`, `Map`, `Decimal`, `UInt`, `Bytes`,
   `format`, `secret`, `Secret`, `resource`, `transaction`, `modifies`, `old`,
   `property`, `std`.

Using any keyword in (1) or (3) as an identifier produces `WOLIO-SYNTAX-001`.
Redeclaring any builtin in (2) or binding `result` produces `WOLIO-RESOLVE-001`.

## Grammar

EBNF: `{x}` repeats, `[x]` is optional, `|` is alternative. Quoted text is literal.
`ident`, `integer`, and `string` use the lexical rules above.

```ebnf
file = "module", ident, ";", { import }, { declaration } ;
import = "import", ident, ";" ;
declaration = [ "pub" ], ( nominal | record | enum | operation )
            | implementation | test ;
nominal = "type", ident, "=", type, ";" ;
record = "record", ident, "{", [ fields ], "}" ;
fields = ident, ":", type, { ",", ident, ":", type } ;
enum = "enum", ident, "{", variant, { ",", variant }, "}" ;
variant = ident, [ "(", type, ")" ] ;
type = name, [ "<", type, { ",", type }, ">" ] ;
name = ident, { "::", ident } ;
operation = "operation", ident, "(", [ fields ], ")", "->", type,
            "{", { clause }, "}" ;
clause = ( "requires" | "ensures" ), expr, ";"
       | ( "effects" | "dependencies" | "errors" ), "[", [ names ], "]", ";"
       | "intent", string, ";" ;
names = name, { ",", name } ;
implementation = "implementation", ident, block ;
block = "{", { "let", ident, "=", expr, ";" }, expr, "}" ;
test = "test", ident, "for", name, block ;
expr = unary, { binary_op, unary } ;
unary = [ "!" | "-" ], primary, { ".", ident } ;
primary = integer | string | "true" | "false" | "(", expr, ")"
        | name, [ "(", [ args ], ")" ]
        | name, "{", [ initializers ], "}"
        | "if", "(", expr, ")", block, "else", block
        | "match", "(", expr, ")", "{", arm, { ",", arm }, "}" ;
args = expr, { ",", expr } ;
initializers = ident, ":", expr, { ",", ident, ":", expr } ;
arm = pattern, "=>", expr ;
pattern = name, [ "(", ident, ")" ] | "true" | "false" ;
binary_op = "||" | "&&" | "==" | "!=" | "<" | "<=" | ">" | ">="
          | "+" | "-" | "*" | "/" | "%" ;
```

Precedence low to high: `||`, `&&`, equality, ordering, `+ -`, `* / %`,
unary, field access. Binary arithmetic/logical operators associate left;
comparison chaining is rejected. Evaluation is left-to-right; `&&`, `||`,
`if`, and `match` evaluate only the selected continuation. Parentheses around
if conditions and match subjects disambiguate record construction from blocks.

## Types and values

- Builtins: `Bool`, signed 64-bit `Int`, UTF-8 `String`, `Unit`, `Option<T>`,
  `Result<T,E>`. `unit()` constructs Unit. Other builtin generics are rejected.
- Checked integer overflow and division by zero produce runtime faults, including
  `MIN / -1`; division truncates toward zero, remainder has the dividend's sign,
  and `MIN % -1` is `0`. The literal magnitude `9223372036854775808` is allowed
  only immediately under unary minus, producing `MIN`. Arithmetic never wraps,
  in any backend or build mode.
- Values are immutable, structurally copied by semantics. Runtime may share
  immutable storage. No references, pointers, assignment, or observable identity.
- `type CustomerId = Int;` creates a nominal type, not an alias. `CustomerId(1)`
  constructs it; `.value` explicitly extracts its representation. Distinct nominal
  types do not coerce. Recursive value-type declarations are rejected in V1.
- Records require exactly their declared named fields; declaration order does not
  affect construction. Enum variants have zero or one payload. Constructors are
  `Status::Ready`, `Status::Failed(reason)`, `Option::None`, `Option::Some(x)`,
  `Result::Ok(x)`, `Result::Err(e)`. Expected types may resolve builtin constructors;
  unresolved type parameters are errors, not dynamic values.
- `+ - * / %` accept Int; ordering accepts Int; equality accepts identical types
  recursively (including records/enums); logical operators accept Bool. No
  implicit conversions, truthiness, null, float, or string concatenation in V1
  (deterministic `format(...)` and `Decimal` are introduced in `CORE_V2.md`).
- Let bindings are immutable and cannot shadow within an operation. A block's
  final expression is its result. Branch results must have the same type.
- Match is permitted on enums, Option, Result, and Bool; one arm per constructor,
  no wildcards/guards, missing and duplicate cases are errors. Payload bindings
  exist only within their arm. Ignoring a Result-valued let binding is an error.

## Modules, calls, and execution

One flat module per file; module names must be unique within the package.
Imports name modules, not paths. `module::symbol` resolves only imported public
declarations; local names resolve lexically. No implicit prelude except builtins.
Operation signatures are explicit; exactly one local implementation is required.
No overloading, dynamic dispatch, closures, loops, or recursive call cycles in V1
(total bounded loops and pure predicates are introduced in `CORE_V2.md`).
Operation calls are the only user-defined computation calls in V1.

All called operations must appear in `dependencies`, including local calls;
constructors and builtin `unit()` are exempt. Effects are upper bounds and
propagate transitively. Missing lists mean empty. Duplicate list clauses are
errors. Builtin `console::print(String) -> Unit` has effect `Console::Write`;
it is the sole V1 I/O operation and must be declared as a dependency. Tests
capture its output. No implicit clock, randomness, environment, or network.

Entry operation `main() -> Unit` or `main() -> Int` is selected by manifest.
`run` prints a returned Int as the result, not as a process exit code. Runtime
faults and contract violations abort the invocation and produce diagnostics.
`check` does not execute programs or imply universal contract correctness.

## Tests and example

Test blocks return Bool: true passes, false/fault fails. Each test names its
target operation. Contract-failure fixtures are host-side tests expecting a
specific diagnostic; source tests cannot catch or suppress contract violations.

```wol
module discount;
operation Clamp(x: Int) -> Int {
    requires x >= 0;
    ensures result >= 0 && result <= x;
    effects [];
    dependencies [];
    intent "Return at most ten units.";
}
implementation Clamp { if (x > 10) { 10 } else { x } }
test boundary for Clamp { Clamp(0) == 0 }
```

## Milestone staging rule (M01 -> M02 -> M03)

- **M01 (`T001`–`T004`):** Parses the full `CORE_V1` grammar and reports all
  semantic/contract obligations as `not_run`.
- **M02 (`T020`–`T024`):** Resolves modules/symbols, checks types, assigns
  `sym_` IDs, and executes programs via `wolio run`. Declarative `effects`,
  `dependencies`, `errors`, and `intent` clauses are parsed and allowed in M02
  (their static set verification is reported `not_run` until `T031`). Only
  operations containing `requires` or `ensures` predicate clauses fail `wolio run`
  at M02 with `WOLIO-FEATURE-001` until `T030` implements runtime predicate guards.
- **M03 (`T030`–`T034`):** Enables full V1 `requires`/`ensures` runtime guards
  (`T030`) and static `errors`/`effects`/`dependencies` set enforcement (`T031`).
