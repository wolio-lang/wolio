# Baseline acceptance

Revision 1.4, 2026-11-10: **READY FOR OWNER REVIEW — NOT YET FROZEN**.

Revision 1.4 (`ADR-008`) purges all 46 superseded historical draft files,
resolves all remaining V1 task open items (`T020`, `T024`), hardens the V1
anti-workaround analyzer (`WOLIO-ANALYZER-001`..`005`), forward-reserves all V2
and V3 keywords in V1, makes task contracts self-contained, and establishes the
non-colliding V2 (`BASELINE-V2.md`, `CORE_V2.md`, `V2-TASKS.md`) and V3
(`BASELINE-V3.md`, `CORE_V3.md`, `V3-TASKS.md`) blueprints with mandatory
inter-version Bug-Fix & Regression Gates (`M08` and `M12`).

Acceptance record to complete after owner review:

- Decision: pending
- Owner/date: pending
- Design revision: 1.4
- Reviewed documentation commit/hash: pending
- Exceptions/amendments: pending

The acceptance record may be committed after the reviewed documentation commit;
its hash identifies the reviewed baseline, avoiding a self-referential commit
hash. No implementation or benchmark validation has occurred yet. Subsequent
semantic changes require an ADR, compatibility/fixture impact, and explicit
owner decision.
