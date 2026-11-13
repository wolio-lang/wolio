`.blueprint/02-specifications/INTERFACE_CONTRACTS.md`
# Shared interface contracts V1–V3

Normative clarification for revision 1.4. These rules connect the language,
package, CLI, and tooling specifications; internal Rust function signatures remain
implementation choices.

## Project inputs

Project root contains `wolio.toml`, `src/**/*.wol`, optional `tests/**/*.wol`,
and (with external dependencies) `wolio.lock`. Source discovery uses sorted
normalized root-relative paths, does not follow directory symlinks, and excludes
`.wolio/`, `.wolio-log/`, `.git/`, build output, and dependency caches. Duplicate
module names in the compiled package fail (`WOLIO-RESOLVE-001`). Test modules
enter `test`/`verify` only.

```toml
[package]
name = "example"
version = "0.1.0"
[language]
version = "1"
[entrypoint]
module = "main"
operation = "main"
```

`[language] version` selects the language profile (`"1"` for `CORE_V1.md`, `"2"`
for `CORE_V2.md`, `"3"` for `CORE_V3.md`). Libraries may omit `[entrypoint]`;
`run` and `build` require it. Entry operations must be public for package
invocation. `init <directory>` creates this manifest, a valid `src/main.wol`
returning `Unit`, and refuses to overwrite existing files. No implicit directory
traversal outside the chosen target.

## CLI and output

All commands accept `--json`; project commands accept `--project <directory>`
(default current directory). `check [file]` additionally accepts one standalone
source for frontend diagnostics; a project is required for imports and execution.
Command availability in V1:
- `M01`: `init`, `check`
- `M02`: `run`
- `M03`: `test`, `verify`, `inspect`, `impact`
- `M04`: `agent`
- `M05`: `build`
- `M07`: `add`, `remove`, `install`, `update`, `list`, `info`, `graph`, `fmt`, `lsp`

Unsupported commands or features for the current milestone/language version
produce a usage error (exit 2) or `WOLIO-FEATURE-001` diagnostic (exit 1), never
a successful no-op. `M01` `check` reports unimplemented semantic obligations as
`not_run`. At `M02`, `effects`, `dependencies`, `errors`, and `intent` clauses are
parsed and permitted in source (with static set verification reported `not_run`
until `M03`); only `requires` and `ensures` predicate clauses fail `wolio run` at
`M02` with `WOLIO-FEATURE-001`.

JSON command envelopes follow [EVIDENCE_MODEL.md](verification/EVIDENCE_MODEL.md):
`schema_version: 1`, `command`, `status` (`"success"` or `"failure"`),
`diagnostics` (array), `evidence` (array), and command-specific data in `result`.
`run` additionally has `output` (captured console string) and `result.value`
(integer or `null` for `Unit`). Human `run` prints console output followed by the
`Int` result and newline, if any. Exit status follows `EVIDENCE_MODEL.md` (`0`
success, `1` source/test/runtime rejection, `2` usage/configuration error, `3`
internal failure), never the returned `Int`. JSON stdout is exactly one object
plus newline; logging is stderr. `agent` and `lsp` use their respective streaming
protocols instead of this single-object envelope.

`inspect [symbol-id]` returns sorted nodes/edges and snapshot; no ID means the
whole graph. `impact --before <project-dir> --after <project-dir>` compares two
explicit snapshots, returns semantic changes, compatibility, and reverse-reachable
callers/tests with edge paths. Neither command writes inputs. `test` runs tests in
stable module/name order with isolated captured output; `verify` also applies the
profile's evidence and analyzer gates. `build` checks before lowering.
`fmt --check` is read-only.

## Names and dependency packages

V1–V3 retain flat module imports (`import <module>;`). At `M07` the locked package
closure exports public modules into the import lookup; colliding module names
across that closure are rejected (`WOLIO-RESOLVE-001`) naming both package
identities. There is no implicit package alias syntax. Imports cannot access
private declarations. Dependency packages are read-only inputs; structured
mutations may change root sources/manifest/lock only within authorized scope.
Dependencies do not supply entrypoints to the root project.

## Canonical hashes and identities

Canonical JSON uses UTF-8, sorted object keys, no insignificant whitespace, no
floats, and preserves array order unless the schema specifies sorting. Hash inputs
are domain-tagged and encoded as a sequence of length-prefixed byte fields
(unsigned 64-bit big-endian lengths), eliminating concatenation ambiguity.
Snapshot fields are sorted relative paths plus bytes for source and explicit
metadata/policy files; absent optional files have an explicit absent marker.
SHA-256 is lowercase hex. Contract normalization ignores comments/spans and
formatting, preserves expression/statement order, resolves names to IDs, and
sorts set-valued clauses only. It never simplifies algebra or reorders predicates.

[SEMANTIC_MODEL.md](SEMANTIC_MODEL.md) defines deterministic declaration keys and
hash domain (`wolio.symbol.v1`). No identity registry (`wolio.ids.json`) is an
input or output, and `inspect --register-ids` is not a supported option. IDs are
available from `M02` without mutation. Rename/move provenance is an explicit pair
of old/new IDs bound to base/result snapshots; read-only comparison reports
removal/addition without inventing that link.

## Lexical, reserved-keyword, and binding clarifications

1. **V1 active keywords (reserved in every position, `WOLIO-SYNTAX-001` if used as identifier):**
   `module`, `import`, `pub`, `type`, `record`, `enum`, `operation`, `requires`,
   `ensures`, `effects`, `dependencies`, `errors`, `intent`, `implementation`,
   `test`, `for`, `let`, `if`, `else`, `match`, `true`, `false`.
2. **V2 & V3 forward-reserved keywords (reserved in V1 lexer to prevent cross-version collision; `WOLIO-SYNTAX-001` if used as identifier):**
   `in`, `with`, `bound`, `invariant`, `predicate`, `List`, `Map`, `Decimal`,
   `UInt`, `Bytes`, `format`, `secret`, `Secret`, `resource`, `transaction`,
   `modifies`, `old`, `property`, `std`.
3. **Reserved builtin names and bindings (`WOLIO-RESOLVE-001` if redeclared or shadowed):**
   `Bool`, `Int`, `String`, `Unit`, `Option`, `Result`, `None`, `Some`, `Ok`,
   `Err`, `unit`, `console`, `result`.
4. Parameters, `let` bindings, and pattern payloads may not shadow visible
   bindings (`WOLIO-RESOLVE-001`); duplicate fields, variants, modules, and
   declarations fail (`WOLIO-RESOLVE-001`).
5. Imports and user call graphs in V1 may not form cycles (`WOLIO-RESOLVE-001`).
   Operations have exactly one implementation in the same module (`WOLIO-RESOLVE-001`).
6. Strings reject unescaped control characters (`U+0000`..`U+001F`) and unpaired
   surrogate escapes; JSON surrogate pairs decode to one Unicode scalar. CRLF and
   LF are whitespace; spans count original zero-based UTF-8 bytes (`start`
   inclusive, `end` exclusive). No trailing commas or block comments.
7. The grammar's unary prefix applies after field selection (`-x.value`). `Int`
   `MIN % -1` is `0`; division `MIN / -1` faults (`WOLIO-RUNTIME-001`).
8. A `Result`-valued binding is handled only by exhaustive `match` or forwarding
   it as a returned expression; assigning, comparing, or merely mentioning it does
   not discharge handling obligations (`WOLIO-ERROR-001`).
