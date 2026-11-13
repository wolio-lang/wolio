`.blueprint/01-architecture/decisions/ADR-006-language-first-baseline.md`
# ADR-006 — Language-first contract-first V1

Status: proposed for revision 1.1 freeze. Date: 2026-11-06.

Context: platform scope was broad while contract grammar remained unresolved.
Owner wants a complete reference and may stop at a useful experiment.

Decision: immutable sequential core, nominal types, explicit errors/transitive
effects, executable predicates, interpreter first, local agent after trial.
BASELINE-V1 and its normative documents define exact behavior.

Alternatives: LLVM first delays feedback; full platform first expands scope;
arbitrary predicates introduce termination/proof machinery too early. Existing
language libraries remain valid benchmark comparators, not assumed inferior.

Consequences: no mutation/concurrency/FFI in V1; runtime checking cost is measured;
no universal proof claim. Amend with compatibility and conformance evidence.
ADR-005 Git package sourcing remains valid.
