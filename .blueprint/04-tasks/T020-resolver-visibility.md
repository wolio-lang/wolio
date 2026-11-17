`.blueprint/04-tasks/T020-resolver-visibility.md`
# T020: Resolve modules, imports, visibility and cycle checks

| Field | Value |
|---|---|
| Milestone | `M02` |
| Priority | `P0` |
| Depends on | T004 |
| Blocks | T021 |
| Acceptance families | SEM (resolution, visibility, cycles) |
| Bounded areas | `crates/wolio-semantic` (resolver submodule), semantic support |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T020.md` |

## Objective
Resolve every name in a discovered project to exactly one declaration or
binding, enforcing module uniqueness, import rules, visibility, binding rules,
and the V1 import/call cycle bans with `WOLIO-RESOLVE-001` diagnostics.

## Authority
- Normative: [CORE_V1.md](../02-specifications/language/CORE_V1.md) (modules, calls, and execution), [INTERFACE_CONTRACTS.md](../02-specifications/INTERFACE_CONTRACTS.md) (project inputs; names; lexical and binding clarifications), [EVIDENCE_MODEL.md](../02-specifications/verification/EVIDENCE_MODEL.md) (code registry)
- Decision: [ADR-008](../01-architecture/decisions/ADR-008-multi-version-anti-workaround-roadmap.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `SEM`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Crate:** `crates/wolio-semantic` (submodule `resolver`, depends on `wolio-parser`, `wolio-ast`, `wolio-diagnostics`).
- **Diagnostic code:** All resolution, duplicate, visibility, shadowing, missing/duplicate implementation, import cycle, and recursive call cycle violations emit `WOLIO-RESOLVE-001`.
- **Reserved builtins & binding names (`WOLIO-RESOLVE-001` if redeclared or shadowed):** `Bool`, `Int`, `String`, `Unit`, `Option`, `Result`, `None`, `Some`, `Ok`, `Err`, `unit`, `console`, `result`.

## Scope
In scope:
- Module table for the compiled package from T004 discovery; `tests/**/*.wol` modules excluded from ordinary compilation.
- Import resolution, qualified `module::symbol`, and lexical local names.
- Binding rules (reserved names, shadowing, duplicates) and 1-to-1 `operation` to `implementation` pairing in the same module.
- Import-cycle and recursive operation call-cycle rejection on the resolved graph (`WOLIO-RESOLVE-001`).

Out of scope (owner):
- Project discovery and CLI envelopes (T004); types and recursive value types (T021); builtin generics and match (T022); persistent `sym_` IDs and query graph (T023); `dependencies` list enforcement (T031); test-module inclusion in `test`/`verify` (T032); package-closure imports (T070).

## Normative requirements
- One flat module per file; module names are unique within the package; duplicate module names fail with `WOLIO-RESOLVE-001` (CORE_V1, INTERFACE_CONTRACTS).
- Imports name modules, not paths. `module::symbol` resolves only imported public declarations; local names resolve lexically; no implicit prelude except builtins; imports cannot access private declarations (`WOLIO-RESOLVE-001`) (CORE_V1, INTERFACE_CONTRACTS).
- Imports may not form cycles, and operation calls may not form direct or indirect recursive call cycles; both emit `WOLIO-RESOLVE-001` (CORE_V1, INTERFACE_CONTRACTS, EVIDENCE_MODEL).
- Builtin type/constructor names and `console` cannot be redeclared; `result` is reserved as a binding name; parameters, `let` bindings, and pattern payloads may not shadow visible bindings; duplicate fields, variants, and declarations fail with `WOLIO-RESOLVE-001` (INTERFACE_CONTRACTS).
- Each operation has exactly one implementation in the same module (`WOLIO-RESOLVE-001`) (CORE_V1, INTERFACE_CONTRACTS).

## Acceptance criteria
Positive and boundary:
- [ ] Imported public symbols resolve across modules; local names resolve lexically
- [ ] Builtins (`Bool`, `Int`, `String`, `Unit`, `Option`, `Result`, `unit`, `console::print`) resolve without an import; nothing else is implicitly visible
- [ ] Test modules do not enter ordinary compilation

Rejection and failure:
- [ ] Duplicate module, declaration, field, and variant names are rejected (`WOLIO-RESOLVE-001`)
- [ ] Private and missing symbols, and qualified use without `import`, are rejected (`WOLIO-RESOLVE-001`)
- [ ] Import cycles and direct/indirect recursive call cycles are rejected (`WOLIO-RESOLVE-001`)
- [ ] Redeclared builtin/`console`, `result` bindings, and variable shadowing are rejected (`WOLIO-RESOLVE-001`)
- [ ] Missing or duplicate operation implementations are rejected (`WOLIO-RESOLVE-001`)

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p wolio-cli -- check --project <sem-fixture-project> --json
```
Evidence: `.execution/reports/T020.md` with the SEM fixture list.

## Dependencies and open items
- Depends on: T004; blocks: T021.
- Open: none.
