# Baseline acceptance

Revision 1.5, 2026-11-14: **READY FOR OWNER REVIEW — NOT YET FROZEN**.

Revision 1.5 addresses industrial vibe-coding and systems architecture critiques:
forward-reserves all systems keywords in V1 (23 keywords), enables canonical Wolio
source payloads in the Agent Protocol to eliminate the JSON-AST paradox, introduces
early scalar boundary/property fuzzing (`wolio verify --fuzz`), rescopes V1 M06 to
a 10-task contract benchmark (200 runs), consolidates crates to 14 by unifying
`crates/wolio-semantic`, pins native codegen to verified textual LLVM IR (`.ll`) +
`clang`, standardizes conformance fixture headers, completes V2 systems capabilities
(turbofish, checked bitwise, exact decimal with rounding, 960-run benchmark), completes
V3 systems capabilities (capability-gated C FFI, fixed-width integers UInt8..UInt32/Int8..Int32,
linear resource acquisition deadlock resolution, and tasks T316–T319), and eliminates
task dual-source hazards.

Acceptance record to complete after owner review:

- Decision: pending
- Owner/date: pending
- Design revision: 1.5
- Reviewed documentation commit/hash: pending
- Exceptions/amendments: pending

The acceptance record may be committed after the reviewed documentation commit;
its hash identifies the reviewed baseline, avoiding a self-referential commit
hash. No implementation or benchmark validation has occurred yet. Subsequent
semantic changes require an ADR, compatibility/fixture impact, and explicit
owner decision.
