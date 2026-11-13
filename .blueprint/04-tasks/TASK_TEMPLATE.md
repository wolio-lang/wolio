`.blueprint/04-tasks/TASK_TEMPLATE.md`
# Tnnn: <short imperative title>

Copy into `Tnnn-<kebab-slug>.md` only after allocating the ID in `task-index.md`
under the rules in `EXECUTION_CONTRACTS.md`. Do not record status in this file.

| Field | Value |
|---|---|
| Milestone | `MXX` |
| Priority | `P0` |
| Depends on | `<task IDs or owner baseline acceptance>` |
| Blocks | `<task IDs or none>` |
| Acceptance families | `<ACCEPTANCE_MATRIX family and cases>` |
| Bounded areas | `<responsibility areas/paths>` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/Tnnn.md` |

## Objective
<One measurable outcome.>

## Authority
- Normative: `<links to documents in DOCUMENT_AUTHORITY>`
- Acceptance: `<ACCEPTANCE_MATRIX family>`, DEFINITION_OF_DONE
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Self-contained implementation context
- `<exact crate path, allowed dependencies, diagnostic codes, and key data structures>`

## Scope
In scope:
- `<deliverable>`

Out of scope (owner):
- `<item>` (`<owner task ID>`)

## Normative requirements
Traceability summary; the cited document wins on any difference.
- `<rule>` (`<source document>`)

## Acceptance criteria
Positive and boundary:
- [ ] `<observable behavior>`

Rejection and failure:
- [ ] `<failure/boundary behavior with diagnostic code>`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
<commands>
```
Evidence: `.execution/reports/Tnnn.md`

## Dependencies and open items
- Depends on: `<task IDs>`; blocks: `<task IDs>`
- Execution prerequisites: `<external inputs or none>`
- Open: none
