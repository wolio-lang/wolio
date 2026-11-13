`.blueprint/05-quality/ACCEPTANCE_MATRIX.md`
# V1–V3 acceptance matrix

Each row is a required conformance fixture family under `tests/conformance/<family>/`,
executed by host-side tests through the real frontend/runtime. Do not add fake
passing tests.

## 1. V1 acceptance families (`M01`–`M07`)

| Family | Positive and boundary evidence | Required rejection/failure evidence | Owner |
|---|---|---|---|
| `FRONT` | All `CORE_V1` grammar productions, Unicode strings, CRLF byte spans, precedence, `-x.value`, `-9223372036854775808`, empty module | Invalid UTF-8/token/escape, unfinished string, malformed declaration, chained comparisons, V1/V2/V3 reserved keywords used as identifiers (`WOLIO-SYNTAX-001`) | `T002`–`T004` |
| `SEM` | Imported public symbols, nominal constructors/`.value`, records, enums, `Option`/`Result`/`unit()`, exhaustive matches | Duplicate/private/missing symbols, import/call cycles (`WOLIO-RESOLVE-001`), nominal mismatch, bad arity, recursive values (`WOLIO-TYPE-001`), missing/duplicate arms (`WOLIO-MATCH-001`), unhandled `Result` (`WOLIO-ERROR-001`) | `T020`–`T022` |
| `ID` | Deterministic `sym_` IDs on formatting/relocation, stable module identity, explicit rename provenance, cache deletion reproduction, local-to-Git migration | Hash collision, duplicate semantic key, package identity change without impact, implicit rename transfer, stale snapshot | `T023` |
| `RUN` | `Unit`/`Int` entry, selected branches, short-circuiting, checked limits (`MAX`, `MIN`, `MIN % -1`), console capture, declarative M02 `effects`/`dependencies` allowed | Overflow, divide zero, `MIN / -1` (`WOLIO-RUNTIME-001`), missing entry, `requires`/`ensures` at M02 (`WOLIO-FEATURE-001`) | `T024` |
| `CONTRACT` | Conjunction/source order, nominal inputs, `Result` postconditions, nested operation guard order | `result` in `requires`, predicate calls/type errors, literal `false` `requires` (`WOLIO-CONTRACT-001`), runtime false `requires` (`002`) / `ensures` (`003`), predicate overflow (`004`) | `T030` |
| `EFFECT` | Propagated error/effect sets, explicitly handled `Result`, unused declared error variant | Undeclared `Err` (`WOLIO-ERROR-001`), hidden transitive effect (`WOLIO-EFFECT-001`), missing dependency (`WOLIO-DEPENDENCY-001`), discarded `Result` | `T031` |
| `EVID` | `pass`/`fail`/`not_run`/`unknown`, deterministic test order, snapshot-bound evidence | Changed input or delete/recreate invalidates evidence, missing reachable-error/boundary tests, false/faulting tests | `T032` |
| `QUERY` | Stable `inspect`, typed reverse paths, identical normalized predicate compatibility | Removed public API, changed nominal/signature/sets; `unknown` non-identical predicate implication | `T033` |
| `TRIAL` | Three runnable examples, `init`/`check`/`run`/`test`/`inspect`/`impact`/`verify`, three change cycles | Wrong implementation passes types but fails `ensures`; documented limitations and raw pilot runs | `T034` |
| `AGENT` | Typed AST mutation, raw-text rejection, `AUTO` and `POST_REVIEW` publication, review record, replay, finalization | Public/unknown contract without approval, tier bypass, self-review, stale/expired/revoked approval, traversal/symlinks (`WOLIO-SCOPE-001`), crash at publication steps, concurrent user edits (`WOLIO-STALE-001`), failed verify | `T040`–`T044` |
| `ANALYZER` | Unconfigured literals pass, test/contract/`intent` exclusions, warning-only verification on `005` | Suppression bypass (`WOLIO-ANALYZER-001`), swallowed `Err` returning constant `Ok` (`002`), unused declared dependency/effect (`003`), forbidden literal or constant-folded expression (`004`), identical arms or test-literal mirroring warning (`005`) | `T044` |
| `NATIVE` | `VerifiedWolioIr` boundary, stable C ABI, release guards, interpreter/native differential corpus | Unverified/ill-typed IR, bad edges/terminators, matching faults and contract clauses, unsupported target | `T050`–`T053` |
| `BENCH` | Corpus/prompts/hashes/versions, 24 × 10 paired runs, raw data and 10,000-resample bootstrap 95% intervals | Timeouts/failures retained, no selective omission, no unsupported improvement claims | `T060`–`T063` |
| `PKG` | Deterministic graph, private Git helper, verified `~/.wolio/cache/`, `--locked --offline` install | Conflicting ranges/cycles, moved tags, altered bytes (`WOLIO-PACKAGE-INTEGRITY-001`), missing offline data, module collision, escaping symlinks/submodules | `T070`–`T071` |
| `TOOL` | Format idempotence/comment/`sym_` preservation, LSP hover/definition/references/rename, clean Linux install | `fmt --check` changes nothing, stale editor version, unsaved buffer isolation, corrupt archive/checksum | `T072`–`T074` |

## 2. V2 & V3 acceptance families (`M08`–`M15`)

| Family | Version | Scope & required positive/negative evidence | Owner |
|---|---|---|---|
| `V2-REGRESSION` | V2 (`M08`) | All V1 `BUG-*` conformance fixtures pass in interpreter and native builds; V1 version-isolation check | `T201`–`T203` |
| `V2-LOOP` | V2 (`M09`) | Total bounded `for..in..with..bound..invariant` over `List<T>` and `a..b`; `WOLIO-CONTRACT-005` (`bound` false) and `006` (`invariant` false on entry/step/exit) | `T204`, `T206` |
| `V2-DATA` | V2 (`M09`) | Immutable `List<T>`, `Map<K,V>`, exact `Decimal` (scale 6), `UInt`, `Bytes`, deterministic `format(...)`; overflow/inexact rejections | `T205`, `T207`, `T208` |
| `V2-PRED` | V2 (`M10`) | Reusable pure `predicate` declarations called in `requires`/`ensures`/`bound`/`invariant`; cycle and operation-call rejection | `T209` |
| `V2-ARCH` | V2 (`M10`) | `[architecture]` allow/deny module dependency rules; forbidden transitive import fails with `WOLIO-ARCH-001` | `T210` |
| `V2-AGENT` | V2 (`M10`) | `protocol_version: 2` `create_module`, `delete_module`, `add_import`, `remove_import` and V2 AST payloads | `T211`–`T212` |
| `V2-NATIVE` / `V2-BENCH` | V2 (`M11`) | Native LLVM parity on loops/collections/Decimal; formatter/LSP V2 support; 24-task algorithmic benchmark | `T213`–`T216` |
| `V3-REGRESSION` | V3 (`M12`) | All V2 `BUG-*` fixtures pass; copy-on-write `List`/`Map` memory and latency benchmarks | `T301`–`T303` |
| `V3-STD` / `V3-SECRET` | V3 (`M13`) | `std::string`, `std::collections`, `std::math`, `std::json`, `std::crypto`; `Secret<T>` leak/literal/format rejection (`WOLIO-SECURITY-001`) | `T304`–`T307` |
| `V3-IO` / `V3-TX` | V3 (`M14`) | `std::time`/`fs`/`net`, `with_mock` test enforcement (`WOLIO-EFFECT-002`), linear `resource` (`WOLIO-RESOURCE-001`), `modifies`/`old(...)`, `transaction` rollback | `T308`–`T311` |
| `V3-FUZZ` / `V3-RELEASE` | V3 (`M15`) | `property` declarations and `wolio verify --fuzz` shrinking counterexamples (`WOLIO-FUZZ-001`); full W01–W15 benchmark & V3 release | `T312`–`T315` |

## 3. Common execution gates

Every Rust work unit runs:
- `cargo fmt --all -- --check`
- `cargo clippy --workspace --all-targets -- -D warnings`
- `cargo test --workspace`
(`T001` additionally runs `cargo check --workspace`).
Keep tests independent of network except explicitly marked Git integration tests
using controlled local repositories. Never require production credentials in CI.
