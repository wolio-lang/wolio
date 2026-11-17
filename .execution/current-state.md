# Current state

2026-11-14: design and preparation only. Revision 1.5 is a freeze candidate.
No Cargo workspace, compiler, interpreter, executable conformance suite, or
benchmark result exists yet.

Revision 1.5 (`PREP-0004`) completed:
- Extended V1 forward-reserved keywords to 23 (adding `with_mock`, `where`, `fn`, `foreign`)
  to guarantee zero cross-version collision for V2/V3 systems capabilities.
- Resolved LLM JSON-AST paradox in `MUTATION_SCHEMA_V1` and `LOCAL_PROTOCOL_V1` by
  allowing canonical Wolio code snippet strings (`code: string`) parsed server-side by
  `wolio-parser` into validated AST diffs, eliminating manual JSON-AST hallucination.
- Added scalar boundary and property fuzzing in M03 (`wolio verify --fuzz`) with
  `WOLIO-VERIFY-002` to defeat AI test-overfitting and hardcoding on scalar types in V1.
- Rescoped V1 M06 evaluation to a 10-task Contract & Protocol Benchmark (200 runs),
  formally scheduling the 24-task algorithmic benchmark matrix for V2 M11 (`T215`).
- Consolidated compiler middle-end into `crates/wolio-semantic` (submodules `resolver`,
  `types`, `graph`, `contracts`, `effects`, `analyzer`), reducing workspace crates from
  19 to 14 and eliminating cyclic/inverted dependency hazards across V1, V2, and V3.
- Pinned native lowering in `wolio-codegen` to verified textual LLVM IR (`.ll`) emission
  and `clang` compilation, eliminating brittle Cargo `llvm-sys` / `inkwell` C++ build risks.
- Standardized conformance fixture header directives (`// @mode:`, `// @expect-status:`,
  `// @expect-code:`) across all `tests/conformance/` fixtures.
- Registered all non-normative overview files in `DOCUMENT_AUTHORITY.md` and designated
  `04-tasks/Tnnn-*.md` as sole normative authority for all tasks.
- Completed V2 blueprint specification (`CORE_V2.md`, `IMPLEMENTATION_SPECIFICATION_V2.md`,
  `T201`–`T216`): Pratt turbofish syntax (`List::empty::<T>()`), checked bitwise operations
  (`bit_and`..`bit_not`), exact decimal with audited rounding modes (`Decimal::mul_round`),
  architecture policy linting (`[architecture]`), and 960-run algorithmic benchmark (`T215`).
- Completed V3 blueprint specification (`CORE_V3.md`, `IMPLEMENTATION_SPECIFICATION_V3.md`,
  `T301`–`T319`): Capability-gated FFI (`foreign type`, `foreign operation`, `[foreign] allowed_libraries`,
  `Foreign::Call`), fixed-width integers (`UInt8`..`UInt32`, `Int8`..`Int32`), linear resource acquisition
  deadlock resolution (`Resource::acquire::<T>(...)` / `Resource::mock::<T>()`), and expanded M14 with
  tasks `T316`–`T319` (19 total V3 tasks).

Next: owner review/acceptance of the Revision 1.5 baseline in
`.execution/baseline-acceptance.md`, then `T001`.
