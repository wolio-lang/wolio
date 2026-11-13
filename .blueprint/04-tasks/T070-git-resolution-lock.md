`.blueprint/04-tasks/T070-git-resolution-lock.md`
# T070: Implement Git package resolution, `wolio.lock` and package CLI

| Field | Value |
|---|---|
| Milestone | `M07` |
| Priority | `P0` |
| Depends on | T044, T053 |
| Blocks | T071 |
| Acceptance families | PKG (graph, conflicts, imports) |
| Bounded areas | package (`crates/wolio-package`), resolver (`crates/wolio-resolver`), CLI (`crates/wolio-cli`) |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T070.md` |

## Objective
Resolve Git-native package graphs deterministically from `wolio.toml`, generate
canonical `wolio.lock` files, expose public dependency modules to the resolver,
and implement `wolio add`, `remove`, `list`, `info` and `graph`.

## Authority
- Normative: [PACKAGE_MANAGER_SPEC.md](../02-specifications/package-manager/PACKAGE_MANAGER_SPEC.md), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md) (names and dependency packages), [SEMANTIC_MODEL.md](../02-specifications/SEMANTIC_MODEL.md), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md)
- Decision: [ADR-005](../01-architecture/decisions/ADR-005-github-package-source.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `PKG`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)

## Scope
In scope:
- Canonical repository identity parsing (`<lowercase-host>/<owner>/<repo>`, no
  `.git` suffix or credentials) and `[module] path` validation.
- Deterministic SemVer/caret version resolution with highest-first backtracking,
  single-version-per-repository enforcement and conflict/cycle diagnostics.
- `wolio.lock` schema version 1 generation and parsing (canonical sorted TOML
  with version, repository, tag, commit, source hash, dependency hash, contract
  hash and compiler compatibility).
- Flat module import lookup across the locked package closure with collision
  rejection naming both package identities.
- `wolio add`, `remove`, `list`, `info` and `graph` CLI commands (`--json`
  supported).

Out of scope (owner):
- Git credential helpers, content-addressed `~/.wolio/cache/`, tamper checks and
  `install`/`update`/`--locked --offline` execution (T071).

## Normative requirements
- Canonical package identity is `<lowercase-host>/<owner>/<repository>` without
  `.git` suffix or credentials; manifest `[module] path` must match the
  repository identity and produces `git:<repo>` package identity in `sym_`
  hashes (PACKAGE_MANAGER_SPEC, SEMANTIC_MODEL).
- Version constraints support exact SemVer or caret ranges only; prereleases
  require an explicit exact version. Only one version per repository identity is
  permitted; conflicting ranges fail with full dependency chains, and dependency
  cycles fail (PACKAGE_MANAGER_SPEC).
- Resolver processes sorted repository identities, trying highest matching
  SemVer tags first with deterministic backtracking, and writes `wolio.lock`
  schema version 1 in canonical sorted TOML (PACKAGE_MANAGER_SPEC).
- The locked package closure exports public modules into flat module import
  lookup without package alias syntax. Colliding module names across the closure
  fail naming both package identities. Dependency packages are read-only, do not
  expose private declarations, and do not supply entrypoints to the root project
  (INTERFACE_CONTRACTS, PACKAGE_MANAGER_SPEC).

## Acceptance criteria
Positive and boundary:
- [ ] Transitive Git dependencies resolve deterministically to identical `wolio.lock` files across runs
- [ ] Public modules from locked dependencies resolve via flat `import <module>;` with `git:` package identities
- [ ] `wolio add`, `remove`, `list`, `info` and `graph` produce deterministic human and `--json` output

Rejection and failure:
- [ ] Conflicting SemVer/caret ranges fail and report both dependency chains; dependency cycles are rejected
- [ ] Module name collisions across the locked package closure fail naming both colliding package identities
- [ ] `[module] path` mismatch, credential-bearing URLs or attempts to import private dependency declarations are rejected

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
Evidence: `.execution/reports/T070.md` with PKG resolution, conflict and module-collision fixtures.

## Dependencies and open items
- Depends on: T044, T053; blocks: T071.
- Open: none.

