`.blueprint/04-tasks/T071-package-integrity-offline.md`
# T071: Enforce Git package integrity, content-addressed cache and offline install

| Field | Value |
|---|---|
| Milestone | `M07` |
| Priority | `P0` |
| Depends on | T070 |
| Blocks | T072 |
| Acceptance families | PKG (tamper, offline); completes all PKG |
| Bounded areas | package (`crates/wolio-package`), CLI (`crates/wolio-cli`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T071.md` |

## Objective
Implement `wolio install` and `wolio update`, content-addressed caching under
`~/.wolio/cache/`, Git credential delegation, cryptographic integrity checks
(`WOLIO-PACKAGE-INTEGRITY-001`) and `--locked --offline` builds.

## Authority
- Normative: [PACKAGE_MANAGER_SPEC.md](../02-specifications/package-manager/PACKAGE_MANAGER_SPEC.md), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `PKG`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- HTTPS and SSH transport enforcement and delegation to the user's configured
  Git credentials/SSH agent/credential helper without storing tokens.
- SHA-256 source hash, contract hash and dependency hash computation and
  verification; rejection of escaping symlinks, Git submodules and build
  scripts.
- Content-addressed cache under `~/.wolio/cache/` with integrity verification on
  every cache hit.
- `wolio install` (including `--locked` and `--offline`) and `wolio update
  [<pkg>]`.

Out of scope (owner):
- Canonical source formatter (T072); LSP server (T073); release distribution
  archives (T074).

## Normative requirements
- Only HTTPS and SSH Git transports are permitted; private repositories use the
  user's configured Git credentials, SSH agent or credential helper; tokens or
  secrets must never be stored in `wolio.toml`, `wolio.lock`, source or
  diagnostics (PACKAGE_MANAGER_SPEC, EVIDENCE_MODEL).
- Source hash is SHA-256 over sorted relative tracked-file paths, executable
  bit, length and bytes at the pinned commit (excluding `.git`). Symlinks
  escaping the package root, Git submodules and package build/install scripts
  are rejected in V1 (PACKAGE_MANAGER_SPEC, DELIVERY_POLICY).
- Contract hash covers normalized public signatures and predicates; dependency
  hash covers sorted child identities and lock entries (PACKAGE_MANAGER_SPEC).
- Ordinary `wolio install` uses the existing `wolio.lock` and never upgrades it;
  `wolio update` explicitly re-resolves. `install --locked --offline` never
  resolves new versions or touches the network, and fails explicitly if required
  objects are missing from `~/.wolio/cache/` (PACKAGE_MANAGER_SPEC,
  DELIVERY_POLICY).
- Any moved tag (pointing to a different commit than locked), altered source or
  cache bytes, or hash mismatch fails with `WOLIO-PACKAGE-INTEGRITY-001`
  (PACKAGE_MANAGER_SPEC, EVIDENCE_MODEL).

## Acceptance criteria
Positive and boundary:
- [ ] `wolio install` populates `~/.wolio/cache/` and verifies commit, source, contract and dependency hashes
- [ ] `wolio install --locked --offline` succeeds with zero network access when cache entries are present and intact
- [ ] Private repository fixtures authenticate through standard Git credential helpers without persisting tokens

Rejection and failure:
- [ ] Moved Git tag, modified repository commit or corrupted `~/.wolio/cache/` entry fails with `WOLIO-PACKAGE-INTEGRITY-001`
- [ ] Missing cache entries under `--offline` fail explicitly without attempting network access
- [ ] Packages containing escaping symlinks, Git submodules or non-HTTPS/SSH transports are rejected

Completion:
- [ ] All ACCEPTANCE_MATRIX `PKG` cases pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Run isolated local Git repository fixtures both online and with network
disconnected, capturing exact `add`, `install`, `update` and `install --locked
--offline` results.
Evidence: `.execution/reports/T071.md` with online/offline Git fixture transcripts and tamper-rejection checks.

## Dependencies and open items
- Depends on: T070; blocks: T072.
- Open: none.

