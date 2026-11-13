`.blueprint/06-project-management/DELIVERY_PLAN.md`
# Complete V1–V3 delivery plan

Task status is execution data in `.execution/task-status.md`. Below is the
frozen multi-generation work breakdown across V1 (`M01`–`M07`), V2 (`M08`–`M11`),
and V3 (`M12`–`M15`).

## 1. V1 Delivery (`M01`–`M07`, `[language] version = "1"`)

| Milestone | Ordered work units | Acceptance and verification |
|---|---|---|
| `M01` — Frontend | `T001` workspace/toolchain; `T002` lexer (with V1+V2+V3 reserved keywords); `T003` grammar/AST; `T004` JSON CLI/`init`/`check` | `cargo fmt --check`; `cargo clippy --workspace -- -D warnings`; `cargo test --workspace`; `FRONT` valid/invalid fixtures |
| `M02` — Executable Core | `T020` resolver/cycles; `T021` nominal/record/enum types; `T022` `Result`/`Option`/`match`; `T023` `sym_` identities/graph; `T024` interpreter/`run` | Execute pure examples; reject bad types/cycles (`WOLIO-RESOLVE-001`); delete cache and reproduce graph; boundary arithmetic faults |
| `M03` — Trial | `T030` predicate checking/guards; `T031` errors/effects/dependencies; `T032` `test`/`verify` runner; `T033` compatibility/`impact`; `T034` trial examples & pilot | All contract conformance cases; success/error/boundary tests; raw 6-task pilot; three sequential change cycles |
| `M04` — Agent | `T040` envelopes/query/replay; `T041` plans/diffs; `T042` scope/approval tiers; `T043` transactional `apply`/journal; `T044` repair/`finalize`/analyzer `001`–`005` | Complete `LOCAL_PROTOCOL_V1` and `ANALYZER` acceptance suites, including interrupted publication and anti-workaround rules |
| `M05` — Native | `T050` typed IR/`VerifiedWolioIr` verifier; `T051` LLVM lowering; `T052` C/runtime ABI; `T053` parity/`build` | Native and interpreter agree on outputs, fault categories, and violated contract clause IDs across all conformance fixtures |
| `M06` — Evaluation | `T060` 24-task corpus; `T061` baseline runners; `T062` 10 paired runs/arm; `T063` bootstrap analysis/report | Published prompts, hashes, model/tool versions, raw runs, and 10,000-resample 95% intervals; no selective omission |
| `M07` — Toolchain | `T070` Git resolution/lock; `T071` SHA-256 integrity/cache/offline; `T072` formatter; `T073` LSP; `T074` Linux x86_64 archive | Moved-tag rejection; `--locked --offline` reproduction; format idempotence; stale editor tests; clean-machine install |

## 2. V2 Delivery (`M08`–`M11`, `[language] version = "2"`)

| Milestone | Ordered work units | Acceptance and verification |
|---|---|---|
| `M08` — V1 Bug-Fix Gate | `T201` triage & `BUG-*` regression suite; `T202` compiler/protocol/LLVM bug fixes; `T203` version-1 isolation lock | Zero open V1 P0/P1 defects; all `tests/conformance/regressions/BUG-*` pass; V1 snapshot hashes unchanged |
| `M09` — Loops & Data | `T204` V2 parser (`for`, `predicate`); `T205` `UInt`/`Decimal`/`Bytes`; `T206` bounded loop verifier & runtime guards (`005`/`006`); `T207` `List<T>`/`Map<K,V>`; `T208` `format(...)` | `V2-LOOP` and `V2-DATA` suites; loop `bound` and `invariant` entry/step/exit failure checks; exact `Decimal` math |
| `M10` — Predicates & Architecture | `T209` reusable pure `predicate`; `T210` `[architecture]` module allow/deny (`WOLIO-ARCH-001`); `T211` Agent Protocol v2 (`create_module`, `add_import`); `T212` V2 loop analyzer rules | `V2-PRED`, `V2-ARCH`, `V2-AGENT` suites; forbidden architecture imports rejected; multi-module agent refactor passes |
| `M11` — V2 Native & Benchmark | `T213` IR/LLVM lowering for loops/collections/Decimal; `T214` V2 formatter & LSP; `T215` 24-task algorithmic benchmark (`W04`/`W07`/`W08`); `T216` V2 release | Interpreter/native parity on all V2 fixtures; published V2 paired benchmark report and Linux x86_64 release |

## 3. V3 Delivery (`M12`–`M15`, `[language] version = "3"`)

| Milestone | Ordered work units | Acceptance and verification |
|---|---|---|
| `M12` — V2 Bug-Fix & Perf Gate | `T301` V2 `BUG-*` regression fixes; `T302` CoW reference-counting optimization for `List`/`Map`; `T303` large-workspace LSP/formatter profiling | Zero open V2 P0/P1 defects; zero memory leaks; CoW collection benchmark passes |
| `M13` — Standard Library & Secrets | `T304` `std::string` & `std::math`; `T305` `std::collections` & `std::json`; `T306` opaque `Secret<T>` (`WOLIO-SECURITY-001`); `T307` `std::crypto` | `V3-STD` and `V3-SECRET` suites; any attempt to print, format, serialize, or extract `.value` from `Secret<T>` fails |
| `M14` — Capability I/O & Transactions | `T308` `std::time`/`fs`/`net` & `[capabilities]`; `T309` mandatory `with_mock` test sandbox (`WOLIO-EFFECT-002`); `T310` linear `resource` (`WOLIO-RESOURCE-001`) & `modifies`/`old(...)`; `T311` `transaction` rollback | `V3-IO` and `V3-TX` suites; unmocked external I/O in tests rejected; fault-injected transaction rollback verified |
| `M15` — Property Fuzzing & V3 Release | `T312` `property` & `wolio verify --fuzz` shrinking counterexamples (`WOLIO-FUZZ-001`); `T313` Agent Protocol v3; `T314` V3 native/LSP/fmt parity; `T315` full W01–W15 benchmark & V3 release | AI test-hardcoding defeated by generated fuzz inputs; full W01–W15 evaluation report and V3 distribution |

## 4. Trial stop point and benchmark protocol

`M03` ships three runnable examples (bounded discount calculation, nominal-ID
lookup returning typed errors, and multi-operation order total), a deliberately
wrong implementation that passes type checking but fails `ensures`, and three
contract change cycles.

Benchmark rules (for `M03` 6-task pilot, `M06` V1 24-task evaluation, `M11` V2
algorithmic evaluation, and `M15` V3 full industrial evaluation):
- Freeze corpus, hidden independent acceptance tests, prompts, budget (3 repair
  attempts, 10 minutes/task), model identifier, sampling settings, tool versions,
  hardware, and pricing before any measured run.
- Hidden tests never enter agent context. Randomize run order. Retain all
  failures and timeouts.
- Compute paired bootstrap 95% confidence intervals over tasks (10,000 resamples).
  An improvement claim requires a positive 95% lower bound for paired correctness
  difference and median cost/time `<= 2x` baseline.
