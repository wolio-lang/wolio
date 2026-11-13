`.blueprint/CHANGELOG.md`
# Blueprint Changelog

> Non-normative history of blueprint revisions. Active rules are defined by
> [DOCUMENT_AUTHORITY.md](DOCUMENT_AUTHORITY.md) and its binding documents.

## Unreleased

### Revision 1.4 — 2026-11-10 (current freeze candidate)

- Incorporated [ADR-008](01-architecture/decisions/ADR-008-multi-version-anti-workaround-roadmap.md):
  - Removed all 46 superseded historical draft files (`LANGUAGE_SPEC.md`, `SCOPE.md`, `WTP_SPEC.md`, 18 crate sketches in `03-implementation/crates/`, 7 milestone drafts in `03-implementation/milestones/`, and `M00-*` reports) to eliminate conflicting syntax and error codes.
  - Reserved all V2 and V3 keywords in the V1 lexer (`CORE_V1.md`, `INTERFACE_CONTRACTS.md`, `T002`) to guarantee zero source collision across V1, V2, and V3.
  - Resolved all remaining task `Open:` items in `T020` (`WOLIO-RESOLVE-001` for import/call cycles) and `T024` (declarative `effects`/`dependencies`/`errors`/`intent` permitted in M02; only `requires`/`ensures` trigger `WOLIO-FEATURE-001` at M02 `run`).
  - Strengthened `WOLIO-ANALYZER-001`..`005` in `DIAGNOSTIC_SPEC.md` and `T044` so `002` (swallowed error returning constant `Ok`), `003` (unused declared dependency/effect padding), `004` (constant-folded forbidden literals), and `005` (identical arms and test-literal mirroring) are concrete, enforceable checks.
  - Added complete, non-colliding blueprints for **V2** ([BASELINE-V2.md](BASELINE-V2.md), [CORE_V2.md](02-specifications/language/CORE_V2.md), [V2-TASKS.md](04-tasks/V2-TASKS.md): `M08`–`M11`, `T201`–`T216`) and **V3** ([BASELINE-V3.md](BASELINE-V3.md), [CORE_V3.md](02-specifications/language/CORE_V3.md), [V3-TASKS.md](04-tasks/V3-TASKS.md): `M12`–`M15`, `T301`–`T315`), including mandatory inter-version Bug-Fix & Regression Gates (`M08` and `M12`).
  - Made task files self-contained and aligned `.execution/baseline-acceptance.md` to Revision 1.4.

### Revision 1.3 — 2026-11-06 (superseded by revision 1.4)

- Incorporated [ADR-007](01-architecture/decisions/ADR-007-deterministic-identity-and-gated-automation.md): removed `wolio.ids.json`, defined deterministic `sym_` SHA-256 declaration identities, required typed-only AST agent mutations, introduced `AUTO`/`POST_REVIEW`/`HUMAN_REQUIRED` risk tiers, made the interpreter the default development backend, and restricted LLVM to `VerifiedWolioIr`.

### Revision 1.2 — 2026-11-06 (superseded by revision 1.3)

- Added authoritative document register, whole-project coverage, shared CLI/manifest/hash contracts, and expanded V1 task contracts `T020`–`T074`.
