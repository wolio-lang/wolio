`.blueprint/04-tasks/T042-scope-approval-tiers.md`
# T042: Enforce path/symbol scope, capability policy and risk-tier approvals

| Field | Value |
|---|---|
| Milestone | `M04` |
| Priority | `P0` |
| Depends on | T041 |
| Blocks | T043 |
| Acceptance families | AGENT (deny, traversal, tier, expiry, revocation) |
| Bounded areas | agent (`crates/wolio-agent`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T042.md` |

## Objective
Enforce deny-by-default path/symbol scope and capability boundaries, derive the
mandatory `AUTO` / `POST_REVIEW` / `HUMAN_REQUIRED` risk tier from semantic
diffs and impact, and validate owner approval records.

## Authority
- Normative: [LOCAL_PROTOCOL_V1.md](../02-specifications/protocol/LOCAL_PROTOCOL_V1.md), [APPROVAL_MATRIX_V1.md](../02-specifications/protocol/APPROVAL_MATRIX_V1.md), [MUTATION_SCHEMA_V1.md](../02-specifications/protocol/MUTATION_SCHEMA_V1.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Decision: [ADR-007](../01-architecture/decisions/ADR-007-deterministic-identity-and-gated-automation.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `AGENT`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Path and symbol scope validation (including reference updates from
  rename/move), `..` traversal rejection and symlink-escape rejection.
- Capability policy enforcement against project policy.
- Deterministic risk-tier calculation (`AUTO`, `POST_REVIEW`, `HUMAN_REQUIRED`)
  from the complete plan diff and transitive impact (`APPROVAL_MATRIX_V1`).
- Validation of owner-authored `HUMAN_REQUIRED` approval records (binding,
  single-use, <= 24h UTC expiry, revocation, non-agent-writable location).

Out of scope (owner):
- Staged workspace verification, lock/CAS publication and crash recovery (T043);
  `POST_REVIEW` disposition ledger, analyzer rules and `finalize` (T044).

## Normative requirements
- Default scope and capabilities are deny; deny overrides allow; empty scope
  matches nothing. Scope checks normalized project-relative paths and `sym_`
  IDs before and after planning; `..` traversal, symlink escapes and out-of-scope
  targets or reference updates fail with `WOLIO-SCOPE-001` (LOCAL_PROTOCOL_V1,
  MUTATION_SCHEMA_V1, EVIDENCE_MODEL).
- Tier precedence is `HUMAN_REQUIRED > POST_REVIEW > AUTO` across all operations
  and transitive impact; the agent cannot supply, lower or override the tier
  (LOCAL_PROTOCOL_V1, APPROVAL_MATRIX_V1).
- `AUTO` covers only implementation edits, test additions, or private
  declaration create/delete with empty `errors`/`effects`/`dependencies` and no
  public, contract, capability, policy or boundary impact (APPROVAL_MATRIX_V1).
- `POST_REVIEW` covers private rename/move and private contract edits whose
  normalized predicates, signatures and `errors`/`effects`/`dependencies` sets
  remain identical with no public impact (LOCAL_PROTOCOL_V1, APPROVAL_MATRIX_V1).
- `HUMAN_REQUIRED` covers all public API/contract/rename/move/delete changes,
  any non-identical (`unknown`) predicate change, any `errors`/`effects`/
  `dependencies` set change, new private operations with non-empty sets,
  capability/security/policy/schema/package-identity/architecture changes, and
  unclassified changes. Deleting tests that are required evidence is rejected
  regardless of tier (LOCAL_PROTOCOL_V1, APPROVAL_MATRIX_V1).
- `HUMAN_REQUIRED` approval records live outside agent-writable scope, bind to
  exact plan hash, diff hash, base snapshot, capabilities, actor scope and
  approver, are valid for one application only, expire within at most 24 hours,
  and honor revocation. Stale or mismatched snapshot/hash bindings fail with
  `WOLIO-STALE-001`; missing, unauthorized, expired, revoked or reused approvals
  fail with `WOLIO-SCOPE-001` (LOCAL_PROTOCOL_V1, APPROVAL_MATRIX_V1,
  EVIDENCE_MODEL).

## Acceptance criteria
Positive and boundary:
- [ ] `AUTO`, `POST_REVIEW` and `HUMAN_REQUIRED` tiers are derived deterministically according to `APPROVAL_MATRIX_V1`
- [ ] Highest tier across a multi-operation plan and its transitive impact wins
- [ ] Valid owner approval bound to exact plan hash, diff hash and base snapshot authorizes one `HUMAN_REQUIRED` plan

Rejection and failure:
- [ ] Out-of-scope symbol/path edits, `..` path traversal and symlink escapes fail with `WOLIO-SCOPE-001`
- [ ] Agent attempts to supply or downgrade approval tier, or write approval records, are rejected
- [ ] Public/unknown contract changes, non-empty sets on new private operations, or set changes without owner approval are rejected
- [ ] Deleting required test evidence is rejected at every tier
- [ ] Expired (> 24h), revoked, reused or hash/snapshot-mismatched approvals are rejected (`WOLIO-SCOPE-001` / `WOLIO-STALE-001`)

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T042.md` with scope, tier and approval fixtures.

## Dependencies and open items
- Depends on: T041; blocks: T043.
- Open: none.

