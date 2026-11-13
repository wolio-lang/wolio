`.blueprint/04-tasks/T210-architecture-module-policy.md`
# T210: Enforce `[architecture]` module dependency allow/deny rules (`WOLIO-ARCH-001`)

| Field | Value |
|---|---|
| Milestone | `M10` |
| Priority | `P0` |
| Depends on | T209 |
| Blocks | T211 |
| Acceptance families | V2-ARCH |
| Bounded areas | `crates/wolio-effects`, `crates/wolio-semantic`, `crates/wolio-verify` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T210.md` |

## Objective
Enforce declarative module-level architecture dependency boundaries (`[architecture]`
`allow_imports` and `deny_imports` in `wolio.toml`) across direct and transitive
module imports, emitting `WOLIO-ARCH-001` whenever an AI agent or developer
introduces a forbidden cross-layer dependency (Taxonomy `W04`).

## Authority
- Normative: [CORE_V2.md](../02-specifications/language/CORE_V2.md) (Section 6: Architecture boundary policy), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md), [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md) (`W04`)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V2.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-ARCH`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **`wolio.toml` `[architecture]` schema (`crates/wolio-semantic`, `crates/wolio-effects`):**
  ```toml
  [architecture]
  allow_imports = [
    { from = "app", to = "domain" },
    { from = "app", to = "infra" },
    { from = "infra", to = "domain" }
  ]
  deny_imports = [
    { from = "domain", to = "infra" },
    { from = "domain", to = "app" }
  ]
  ```
- **Matching & precedence semantics:**
  - `from` and `to` match either an exact module name (e.g., `"domain"`) or a prefix pattern `"domain_*"` (matching any module whose name starts with `"domain_"`).
  - **Deny overrides allow:**
    1. If any rule in `deny_imports` matches a direct import `M_a -> M_b` OR a **transitive import path** `M_a -> ... -> M_b`, the import in `M_a` is rejected with `WOLIO-ARCH-001`.
    2. If `allow_imports` is present and non-empty, every direct `import M_b;` in a production module `M_a` must match at least one `{ from, to }` entry in `allow_imports`; otherwise it is rejected with `WOLIO-ARCH-001`.
    3. If `[architecture]` is omitted from `wolio.toml`, all acyclic module imports are allowed.
  - Test modules (`tests/**/*.wol`) may import any public module under test unless explicitly matched by a `deny_imports` rule.
- **Snapshot & Evidence integration:**
  - Canonical normalized `[architecture]` rules are hashed into the project snapshot hash (`wolio-semantic`).
  - `wolio check` and `wolio verify` emit a `static` obligation record (`clause: "architecture_policy"`) with status `pass` or `fail`.
  - Diagnostic `WOLIO-ARCH-001` includes `violation: { "from_module": ..., "to_module": ..., "path": [...] }` and points its span to the offending `import` declaration.

## Scope
In scope:
- Parsing and validating `[architecture]` `allow_imports` and `deny_imports` in `wolio.toml`.
- Including `[architecture]` policy in the semantic snapshot hash and `wolio impact` compatibility checks (any change to `[architecture]` is classified as `HUMAN_REQUIRED`).
- Direct and transitive module import checking in `wolio-effects` emitting `WOLIO-ARCH-001`.

Out of scope (owner):
- Agent Protocol v2 `add_import` / `create_module` operations consuming this check (T211).

## Normative requirements
- `deny_imports` overrides `allow_imports`; both direct imports (`A -> B`) and transitive import chains (`A -> C -> B`) that violate a deny rule fail static checking with `WOLIO-ARCH-001` (CORE_V2, ACCEPTANCE_MATRIX).
- When `allow_imports` is specified, any direct production module import not covered by an allow rule fails with `WOLIO-ARCH-001` (CORE_V2).
- Modifying `[architecture]` in `wolio.toml` changes the project policy snapshot hash and requires `HUMAN_REQUIRED` approval (APPROVAL_MATRIX_V1, CORE_V2).

## Acceptance criteria
Positive and boundary:
- [ ] Projects without `[architecture]` or with imports satisfying `allow_imports` and `deny_imports` pass `wolio check` and record `architecture_policy: pass`
- [ ] Prefix rules (`from = "domain_*"`, `to = "infra_*"`) match module families deterministically

Rejection and failure:
- [ ] Direct import matching `deny_imports` (`domain` importing `infra`) fails with `WOLIO-ARCH-001`
- [ ] Transitive import bypassing a layer (`domain -> shared -> infra` when `domain -> infra` is denied) fails with `WOLIO-ARCH-001` reporting the full cycle-free chain `["domain", "shared", "infra"]`
- [ ] Direct import omitted from a configured `allow_imports` list fails with `WOLIO-ARCH-001`
- [ ] Malformed `[architecture]` entries in `wolio.toml` fail with exit code `2`

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T210.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T210.md` with `V2-ARCH` direct and transitive violation fixtures.

## Dependencies and open items
- Depends on: T209; blocks: T211.
- Open: none.

