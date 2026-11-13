`.blueprint/06-project-management/AI_BUG_TAXONOMY.md`
# AI Failure and Anti-Workaround Taxonomy (W01–W15)

**Status:** Normative benchmark classification and compiler defense mapping across V1, V2, and V3.

Every AI failure mode below is mapped to the exact Wolio language/compiler
mechanism that prevents or catches it and the release version (`V1`, `V2`, `V3`)
where it enters the benchmark corpus.

| ID | AI Failure / Workaround ("Akal-Akalan") | Example Defect | Wolio Prevention Mechanism & Diagnostic | Benchmark Stratum |
|---|---|---|---|---|
| `W01` | Wrong domain type | Pass `UserId` where `OrderId` is required | Nominal `type` declarations without implicit coercion (`WOLIO-TYPE-001`) | **V1** (`M03`/`M06`) |
| `W02` | Swallowed or ignored error | Discard a failed `Result` or match `Err(_)` and return dummy `Ok(0)` | Mandatory `Result` handling (`WOLIO-ERROR-001`) + anti-swallowing analyzer (`WOLIO-ANALYZER-002`) | **V1** (`M03`/`M06`) |
| `W03` | Hidden dependency or padded list | Call an undeclared operation or blindly pad `dependencies` with unused names | Explicit `dependencies [...]` check (`WOLIO-DEPENDENCY-001`) + anti-padding check (`WOLIO-ANALYZER-003`) | **V1** (`M03`/`M06`) |
| `W04` | Wrong dependency direction | Domain module imports infrastructure/adapter module | `[architecture]` allow/deny module import rules (`WOLIO-ARCH-001`) | **V2** (`M11`) |
| `W05` | Missing test or hardcoded test mirror | Omit boundary/error test, or hardcode `if (x == 42)` to pass a specific unit test | Reachable-error test gate (`T032`), forbidden constant folding (`WOLIO-ANALYZER-004`), test-literal mirroring warning (`005`), and V3 property fuzzing (`WOLIO-FUZZ-001`) | **V1** (`M06`) & **V3** (`M15`) |
| `W06` | Incorrect boundary/edge case | Negative quantity or overflow accepted in calculation | Checked 64-bit `Int` (`WOLIO-RUNTIME-001`) + runtime `requires`/`ensures` guards (`WOLIO-CONTRACT-002`/`003`) | **V1** (`M03`/`M06`) |
| `W07` | Infinite loop or broken loop state | Off-by-one loop bound or corrupted accumulator in collection processing | Total bounded `for..in..with` loops requiring explicit `bound` (`WOLIO-CONTRACT-005`) and step-checked `invariant` (`WOLIO-CONTRACT-006`) | **V2** (`M11`) |
| `W08` | Wrong business rule / float rounding | Apply discount to ineligible customer or lose cents in IEEE-754 float math | `ensures` postconditions + reusable pure `predicate`s + exact 128-bit fixed-scale `Decimal` | **V1** (`M06`) & **V2** (`M11`) |
| `W09` | Resource aliasing conflict | Mutate the same resource handle through two aliased arguments | Linear non-aliasing `resource` handle parameters (`WOLIO-RESOURCE-001`) | **V3** (`M15`) |
| `W10` | Partial state corruption on error | Mutate account balance before a later step fails without rollback | `transaction (res) { ... }` blocks + `modifies [res]` + `old(...)` error-branch postconditions | **V3** (`M15`) |
| `W11` | Secret leakage or unvalidated input | Log an API key in `format`/`print` or pass unvalidated input as domain type | Opaque `Secret<T>` (`WOLIO-SECURITY-001`) + smart-constructor `predicate` provenance | **V3** (`M15`) |
| `W12` | Silent contract weakening | Weaken `requires`/`ensures` or widen `effects`/`errors` to make a broken body pass | `unknown`/public contract changes force `HUMAN_REQUIRED` approval (`APPROVAL_MATRIX_V1.md`, `WOLIO-SCOPE-001`) | **V1** (`M04`/`M06`) |
| `W13` | Live external I/O in tests | Unit test hits real clock, filesystem, or network | Mandatory hermetic `with_mock` handlers for non-console effects in tests (`WOLIO-EFFECT-002`) | **V3** (`M15`) |
| `W14` | Undeclared transitive side effect | Helper operation performs I/O not declared by its caller | Static transitive `effects [...]` upper-bound checking (`WOLIO-EFFECT-001`) | **V1** (`M03`/`M06`) |
| `W15` | Out-of-scope collateral regression | Agent modifies unrelated module or overwrites concurrent user edit | Path/symbol scope enforcement (`WOLIO-SCOPE-001`) + CAS snapshot check (`WOLIO-STALE-001`) + `impact` graph | **V1** (`M04`/`M06`) |

Each benchmark run records the failure ID (`W01`–`W15`), prompt, outcome, repair
iterations, token count, wall-clock time, regressions, and security impact.
