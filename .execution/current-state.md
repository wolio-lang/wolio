# Current state

2026-11-10: design and preparation only. Revision 1.4 is a freeze candidate.
No Cargo workspace, compiler, interpreter, executable conformance suite, or
benchmark result exists yet.

Revision 1.4 (`PREP-0003`, `ADR-008`) completed:
- Removed 46 superseded historical draft files (`LANGUAGE_SPEC.md`, `SCOPE.md`,
  `WTP_SPEC.md`, 18 crate sketches in `03-implementation/crates/`, 7 milestone
  drafts in `03-implementation/milestones/`, and `M00-*` reports) so zero
  conflicting syntax or error codes remain in `.blueprint/`.
- Forward-reserved all V2 and V3 keywords in the V1 lexer (`CORE_V1.md`,
  `INTERFACE_CONTRACTS.md`, `T002`) so V1 programs never collide with V2 or V3.
- Resolved all V1 task `Open:` items (`T020` cycle diagnostic `WOLIO-RESOLVE-001`;
  `T024` M02 vs M03 contract clause rule) and hardened `WOLIO-ANALYZER-001`..`005`
  (`002` swallowed error returning constant `Ok`, `003` unused dependency/effect
  padding, `004` constant-folded forbidden literals, `005` duplicate arms and
  test-literal mirroring).
- Established complete, non-colliding blueprints for **V2** (`BASELINE-V2.md`,
  `CORE_V2.md`, `IMPLEMENTATION_SPECIFICATION_V2.md`, `V2-TASKS.md`, and 16
  standalone self-contained task contracts `T201`–`T216` across `M08`–`M11`)
  and **V3** (`BASELINE-V3.md`, `CORE_V3.md`, `IMPLEMENTATION_SPECIFICATION_V3.md`,
  `V3-TASKS.md`, and 15 standalone self-contained task contracts `T301`–`T315`
  across `M12`–`M15`), including mandatory Bug-Fix & Regression Gates (`M08` and `M12`).
- Aligned `.execution/baseline-acceptance.md` to Revision 1.4. Actual checks are
  recorded in `reports/PREP-0003.md`.

Next: owner review/acceptance of the Revision 1.4 baseline in
`.execution/baseline-acceptance.md`, then `T001`.
