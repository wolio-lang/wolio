`.blueprint/07-ai-context/ENGINEERING_RULES.md`
# Engineering rules

Read DOCUMENT_AUTHORITY and BASELINE-V1, then task-relevant specifications.
Use the task index and execution ledger; do not infer readiness from an old report.
T001 pins Rust; M05 pins LLVM. Benchmark access is needed only by benchmark tasks.

Implement the smallest complete assigned work unit, obey dependency direction,
preserve user work, and run common checks plus the assigned acceptance families.
Compiler/runtime errors become diagnostics, not host panics or fabricated success.
Unknown proof stays unknown; unavailable checks stay not_run. Never weaken a
contract to make an implementation pass without an explicit accepted amendment.

Frozen observable semantics require an ADR with old/new behavior, rationale,
compatibility, affected tasks/fixtures and owner decision. Internal refactors may
proceed within task scope when observable behavior and dependencies are preserved.
Record progress and real evidence in `.execution/`; do not rewrite preparation
reports as implementation evidence. Commit/push only when explicitly requested.
