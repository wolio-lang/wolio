# ADR-007 — Deterministic identity and gated automation

**Status:** Proposed for owner acceptance with blueprint revision 1.3

## Decision

V1 removes `wolio.ids.json`. Declaration identity is deterministic and defined
by `SEMANTIC_MODEL.md`; mutation uses typed AST operations only; approval uses
`AUTO`, `POST_REVIEW`, and `HUMAN_REQUIRED` tiers; the interpreter is the default
development backend; and native codegen accepts only `VerifiedWolioIr`.

The analyzer's V1 hard rules reject semantic suppression, swallowed errors, and
undeclared effects/dependencies. Heuristic suspicion remains distinguishable
from proven illegality and is represented through the diagnostic schema.

## Consequences

Identity registries and registration merge conflicts disappear. Rename/move
lineage is explicit provenance rather than inferred identity transfer. Protocol,
M02 identity, M04 approval, M05 IR, and acceptance fixtures evolve together.
Verification proves technical obligations but not business intent; human approval
therefore remains mandatory for public or unknown contract and security changes.
