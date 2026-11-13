`.blueprint/04-tasks/T023-semantic-identity-graph.md`
# T023: Compute deterministic identities, snapshot graph and provenance model

| Field | Value |
|---|---|
| Milestone | `M02` |
| Priority | `P0` |
| Depends on | T022 |
| Blocks | T024 |
| Acceptance families | ID |
| Bounded areas | semantic, CLI (diagnostic targets) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T023.md` |

## Objective
Assign deterministic `sym_` declaration IDs and clause identities, build the
sorted typed semantic graph bound to a snapshot hash, and define the change
provenance representation, reproducibly from source alone.

## Authority
- Normative: [SEMANTIC_MODEL.md](../02-specifications/SEMANTIC_MODEL.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md) (canonical hashes and identities)
- Decision: [ADR-007](../01-architecture/decisions/ADR-007-deterministic-identity-and-gated-automation.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `ID` (including the additional ID fixtures), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope: identity hashing, package identity derivation, clause identity,
typed nodes/edges, snapshot hashing, rebuildable cache, provenance data model,
IDs as diagnostic `target`.

Out of scope (owner):
- `inspect` and `impact` commands and compatibility (T033; commands are M03).
- Evidence binding/invalidation (T032); recording provenance for mutations
  (T041, T043); Git repository identity resolution (T070).

## Normative requirements
- ID = `sym_` + lowercase SHA-256 hex over length-prefixed fields (unsigned
  64-bit big-endian lengths): domain `wolio.symbol.v1`, package identity, module
  identifier, declaration kind, symbol path. Kinds: `nominal`, `record`, `enum`,
  `operation`, `implementation`, `test`. No path, signature, source bytes or
  version enters the hash (SEMANTIC_MODEL, INTERFACE_CONTRACTS).
- Package identity: `git:` + canonical repository identity when `module.path`
  exists, else `local:` + manifest package name. Standalone check has no package
  identity and null targets (SEMANTIC_MODEL).
- No `wolio.ids.json`, UUIDs, registration or `inspect --register-ids`
  (SEMANTIC_MODEL, INTERFACE_CONTRACTS, ADR-007).
- Formatting, relocation, implementation edits and file moves preserving module
  declarations preserve IDs; rename or move between modules changes IDs;
  read-only comparison never infers lineage (SEMANTIC_MODEL).
- Duplicate semantic keys fail resolution; distinct keys with the same hash fail
  checking (SEMANTIC_MODEL).
- Clause identity = operation ID + normalized clause hash + occurrence index;
  normalization ignores comments/spans/formatting, keeps order, resolves names
  to IDs, sorts set-valued clauses only (SEMANTIC_MODEL, INTERFACE_CONTRACTS).
- Nodes and edges `imports`, `calls`, `depends_on`, `returns`, `uses_type`,
  `has_contract`, `has_effect`, `tests`; queries sort IDs and edges and carry the
  snapshot hash; snapshots hash sorted relative paths and bytes, manifest,
  lockfile and policy with explicit absent markers; caches are excluded
  (SEMANTIC_MODEL, INTERFACE_CONTRACTS).
- Change provenance holds intent, actor, base/result hashes, approved diff,
  affected IDs and evidence references; rename/move links are explicit old/new
  pairs bound to base/result snapshots (SEMANTIC_MODEL, INTERFACE_CONTRACTS).

## Acceptance criteria
Positive and boundary:
- [ ] IDs are identical after reformatting, checkout relocation and module-preserving file moves
- [ ] Module identity is stable; fixed identity maps are used in tests
- [ ] Deleting the cache reproduces an identical graph and snapshot hash
- [ ] Delete then recreate at the same semantic key yields the same ID with a different snapshot
- [ ] Local-to-Git package identity migration changes every declaration ID

Rejection and failure:
- [ ] Duplicate semantic keys and forced hash collisions are rejected
- [ ] Rename is reported as removal plus addition; no implicit lineage transfer
- [ ] Stale cache or graph data bound to another snapshot is never reused

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
Evidence: `.execution/reports/T023.md` with ID fixtures and hash vectors.

## Dependencies and open items
- Depends on: T022; blocks: T024.
- Hand-off: the "package identity change without impact" and evidence
  invalidation cases are completed by T033 and T032 using these identities.
- Open: none.

