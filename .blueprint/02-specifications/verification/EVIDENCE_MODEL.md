`.blueprint/02-specifications/verification/EVIDENCE_MODEL.md`
# Verification evidence and diagnostic registry V1–V3

Each obligation records `id`, `symbol`, `clause`, `source_hash`, `tool_version`,
`method`, `status`, and related diagnostic/test IDs. Methods are `static`,
`runtime_guard`, `test`, or `review` (plus `property_fuzz` in V3). Status is
`pass`, `fail`, `not_run`, or `unknown`. A runtime guard installed successfully
is reported as `runtime_guard`, never as universal static proof that all
executions pass. Test success applies only to recorded inputs. Review cannot
replace a failed language check. Reports preserve this distinction in JSON and
human output.

`check` runs syntax, resolution, types, contract well-formedness, errors,
effects, and dependencies. `test` runs source tests with runtime guards enabled.
`verify` combines `check`, `test`, analyzer rules, and profile evidence gates,
binding results to the exact source, manifest, lockfile, compiler, and policy
hashes. Missing required evidence fails finalization. Unsupported checks are
`not_run`, never silently `pass`. Changed inputs invalidate evidence.

Diagnostics retain [DIAGNOSTIC_SPEC.md](DIAGNOSTIC_SPEC.md)'s shape, with
optional `obligation`, `related`, `evidence`, and `snapshot` fields. Source
spans are zero-based UTF-8 byte offsets, `start` inclusive / `end` exclusive;
paths are normalized relative to project root. `target` is `null` before
identity assignment (`M01` and standalone file checks). Sort by
`(path, start, code, target)`. Version 1 readers ignore unknown optional fields;
changed field meaning requires a major schema revision. No source secrets or
full runtime values may appear in diagnostics.

## Diagnostic code registry (V1 active + V2/V3 reserved)

| Code | Version | Phase / Owner | Meaning |
|---|---|---|---|
| `WOLIO-SYNTAX-001` | V1 | M01 (`T002`, `T003`) | Invalid UTF-8, token, escape, literal range, reserved keyword use, or syntax |
| `WOLIO-RESOLVE-001` | V1 | M02 (`T020`, `T023`) | Missing, duplicate, private, or shadowed symbol; duplicate module; missing/duplicate implementation; import cycle; recursive call cycle; hash collision |
| `WOLIO-TYPE-001` | V1 | M02 (`T021`, `T022`) | Incompatible types, bad arity, nominal mismatch, recursive value type, or invalid generic application |
| `WOLIO-MATCH-001` | V1 | M02 (`T022`) | Missing or duplicate `match` arm, or invalid match subject |
| `WOLIO-ERROR-001` | V1 | M02 (`T022`) / M03 (`T031`) | Unhandled/discarded `Result` binding, or undeclared/invalid `errors` variant |
| `WOLIO-EFFECT-001` | V1 | M03 (`T031`) | Undeclared direct or transitive effect |
| `WOLIO-DEPENDENCY-001` | V1 | M03 (`T031`) | Undeclared operation call in `dependencies` |
| `WOLIO-CONTRACT-001` | V1 | M03 (`T030`) | Malformed contract predicate (`result` in `requires`, non-`Bool`, operation call in predicate, or literal `false` in `requires`) |
| `WOLIO-CONTRACT-002` | V1 | M03 (`T030`) | Runtime `requires` precondition violation (caller fault) |
| `WOLIO-CONTRACT-003` | V1 | M03 (`T030`) | Runtime `ensures` postcondition violation (implementation fault) |
| `WOLIO-CONTRACT-004` | V1 | M03 (`T030`) | Arithmetic fault (overflow, `/ 0`, `MIN / -1`) during contract predicate evaluation |
| `WOLIO-RUNTIME-001` | V1 | M02 (`T024`) | Runtime arithmetic fault (overflow, `/ 0`, `MIN / -1`) in operation/test body |
| `WOLIO-FEATURE-001` | V1 | M01–M07 | Unsupported command, profile, target, or higher-version (`V2`/`V3`) construct |
| `WOLIO-SCOPE-001` | V1 | M04 (`T042`) | Unauthorized path/symbol scope, `..`/symlink escape, tier bypass, or invalid/expired/revoked approval |
| `WOLIO-STALE-001` | V1 | M04 (`T040`, `T043`) | Stale `base_snapshot`, approval binding mismatch, or concurrent workspace edit |
| `WOLIO-PACKAGE-INTEGRITY-001` | V1 | M07 (`T071`) | Moved Git tag, modified commit/source/cache bytes, or hash mismatch |
| `WOLIO-ANALYZER-001` | V1 | M04 (`T044`) | Attempted diagnostic suppression or verification bypass (`error`) |
| `WOLIO-ANALYZER-002` | V1 | M04 (`T044`) | Swallowed `Result::Err` returning constant/literal `Result::Ok` (`error`) |
| `WOLIO-ANALYZER-003` | V1 | M04 (`T044`) | Declared `dependencies` or `effects` entry unused by implementation (`error`) |
| `WOLIO-ANALYZER-004` | V1 | M04 (`T044`) | Forbidden literal or constant-folded expression in production code (`error`) |
| `WOLIO-ANALYZER-005` | V1 | M04 (`T044`) | Identical normalized `match` arms or test-input literal mirroring in production branch (`warning`) |
| `WOLIO-CONTRACT-005` | V2 | M09 (`T206`) | Runtime `for` loop `bound` predicate violation |
| `WOLIO-CONTRACT-006` | V2 | M09 (`T206`) | Runtime `for` loop `invariant` predicate violation |
| `WOLIO-ARCH-001` | V2 | M10 (`T210`) | Module import violates `[architecture]` allow/deny policy |
| `WOLIO-SECURITY-001` | V3 | M13 (`T306`) | `Secret<T>` literal construction, `.value` extraction, formatting, printing, or unauthorized sink |
| `WOLIO-EFFECT-002` | V3 | M14 (`T309`) | Unmocked external I/O effect (`Clock`, `Fs`, `Net`, `Db`) inside `test` or `property` |
| `WOLIO-RESOURCE-001` | V3 | M14 (`T310`) | Aliased `resource` handle or missing `modifies` declaration |
| `WOLIO-FUZZ-001` | V3 | M15 (`T312`) | Contract/property fuzzing counterexample discovered (`wolio verify --fuzz`) |

New codes are additive and never reuse old meanings. Repairs are suggestions,
not executable authority; adding an effect, error, or weakening a guarantee is a
contract change. Exit codes: `0` command success, `1` source/test/runtime
rejection, `2` usage/configuration error, `3` internal tool failure. JSON output
is one object with `schema_version`, `command`, `status`, `diagnostics`, and
`evidence` arrays; program output is a separate field for `run` and never
corrupts JSON stdout.
