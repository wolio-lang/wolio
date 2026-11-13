`.blueprint/03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md`
# Wolio implementation specification V1 (`M01`–`M07`)

Status: revision 1.4 freeze candidate. This document is the authoritative
reference for workspace crate boundaries, dependency direction, and compiler
pipeline staging for Wolio V1 (`[language] version = "1"`, milestones `M01`–`M07`,
tasks `T001`–`T074`). For V2 (`M08`–`M11`, `T201`–`T216`) and V3 (`M12`–`M15`,
`T301`–`T315`) crate extensions, data representations, and runtime ABI additions,
see [IMPLEMENTATION_SPECIFICATION_V2.md](IMPLEMENTATION_SPECIFICATION_V2.md) and
[IMPLEMENTATION_SPECIFICATION_V3.md](IMPLEMENTATION_SPECIFICATION_V3.md).
All earlier historical crate and milestone sketch files have been removed.

## 1. Compiler and tooling pipeline

```text
source (.wol)
  -> wolio-lexer (tokens + zero-based UTF-8 byte spans; V1+V2+V3 reserved keywords)
  -> wolio-parser / wolio-ast (concrete syntax -> spanned AST)
  -> wolio-resolver (modules, imports, visibility, binding rules, cycle rejection)
  -> wolio-semantic (deterministic sym_ IDs, clause hashes, typed graph, snapshots)
  -> wolio-types (nominals, records, enums, Option/Result, exhaustive match, Result handling)
  -> wolio-contracts (requires/ensures/bound/invariant typing & predicate purity)
  -> wolio-effects (errors, transitive effects, explicit dependencies, [architecture] edges)
  -> wolio-analyzer (anti-workaround rules WOLIO-ANALYZER-001..005, constant folding)
  -> wolio-verify (test runner, evidence aggregation, inspect/impact compatibility, V3 fuzzing)
  -> wolio-runtime (interpreter execution, checked math, contract guards, native C ABI)
  -> wolio-ir (typed SSA-like IR + structural verifier -> opaque VerifiedWolioIr)
  -> wolio-codegen (pinned LLVM lowering accepting only VerifiedWolioIr)
```

Tooling consumers of the shared semantic model:
- `wolio-agent`: local JSONL transport, typed AST plans/diffs, scope/tier approvals, CAS/journal recovery.
- `wolio-package`: Git tag/commit resolution, `wolio.lock`, SHA-256 cache (`~/.wolio/cache/`), `--locked --offline`.
- `wolio-format`: canonical 4-space LF comment-preserving formatter verifying `sym_` ID preservation.
- `wolio-lsp`: stdio JSON-RPC server with in-memory unsaved-buffer overlay and version-checked rename.
- `wolio-cli`: thin command-line composition layer (never implements language semantics).

## 2. Canonical crate boundaries and milestone creation schedule

Create crates only when their milestone begins (`T001` creates only the `M01` crates):

| Crate | Created in | Responsibility | Allowed workspace dependencies |
|---|---|---|---|
| `crates/wolio-diagnostics` (or module in `wolio-ast`) | `M01` (`T001`) | Shared `Span`, `Diagnostic`, `EvidenceRecord`, code registry, canonical JSON helpers | none (leaf data crate) |
| `crates/wolio-ast` | `M01` (`T001`) | Spanned AST nodes for declarations, contracts, expressions, types, patterns | `wolio-diagnostics` |
| `crates/wolio-lexer` | `M01` (`T001`/`T002`) | UTF-8 validation, tokenization, V1/V2/V3 keyword reservation, string escape validation | `wolio-diagnostics` |
| `crates/wolio-parser` | `M01` (`T001`/`T003`) | Recursive-descent/Pratt parser for `CORE_V1` (and `CORE_V2`/`CORE_V3` by version) | `wolio-lexer`, `wolio-ast`, `wolio-diagnostics` |
| `crates/wolio-resolver` | `M02` (`T020`) | Module discovery table, import resolution, visibility, shadowing, import/call cycle checks | `wolio-ast`, `wolio-diagnostics` |
| `crates/wolio-semantic` | `M02` (`T021`–`T023`) | Interned types, deterministic `sym_` and clause hashes, sorted graph, `inspect`/`impact` | `wolio-ast`, `wolio-resolver`, `wolio-diagnostics` |
| `crates/wolio-types` | `M02` (`T021`–`T022`) | Type checking, nominal isolation, exhaustive `match`, `Result`-handling analysis | `wolio-ast`, `wolio-resolver`, `wolio-semantic`, `wolio-diagnostics` |
| `crates/wolio-runtime` | `M02` (`T024`) | Interpreter evaluator, checked arithmetic, runtime contract guards (`T030`), native C ABI (`T052`) | `wolio-ast`, `wolio-semantic`, `wolio-diagnostics` (NEVER `wolio-codegen` or LLVM) |
| `crates/wolio-contracts` | `M03` (`T030`) | Static checking of `requires`/`ensures` (and V2 `predicate`/`bound`/`invariant`) | `wolio-ast`, `wolio-semantic`, `wolio-types`, `wolio-diagnostics` |
| `crates/wolio-effects` | `M03` (`T031`) | Static `errors` subset, transitive `effects`, explicit `dependencies` (and V2 `[architecture]`) | `wolio-ast`, `wolio-semantic`, `wolio-types`, `wolio-contracts`, `wolio-diagnostics` |
| `crates/wolio-verify` | `M03` (`T032`) | Test discovery/runner, obligation evidence binding, `verify` gates (and V3 property fuzzing) | `wolio-semantic`, `wolio-types`, `wolio-contracts`, `wolio-effects`, `wolio-runtime`, `wolio-diagnostics` |
| `crates/wolio-analyzer` | `M04` (`T044`) | Anti-workaround rules `WOLIO-ANALYZER-001`..`005` (and constant-folding literal check) | `wolio-ast`, `wolio-semantic`, `wolio-types`, `wolio-effects`, `wolio-diagnostics` |
| `crates/wolio-agent` | `M04` (`T040`–`T044`) | Local JSONL server, `MUTATION_SCHEMA` validation, AST renderer, diffs, scope/approvals, CAS journal | `wolio-parser`, `wolio-semantic`, `wolio-verify`, `wolio-analyzer`, `wolio-diagnostics` |
| `crates/wolio-ir` | `M05` (`T050`) | Typed SSA-like IR lowering and structural verifier producing opaque `VerifiedWolioIr` | `wolio-semantic`, `wolio-types`, `wolio-contracts`, `wolio-diagnostics` (NEVER LLVM) |
| `crates/wolio-codegen` | `M05` (`T051`) | Pinned LLVM backend accepting only `VerifiedWolioIr` and linking with `wolio-runtime` | `wolio-ir`, `wolio-runtime`, `wolio-diagnostics` |
| `crates/wolio-package` | `M07` (`T070`–`T071`) | Git SemVer resolver, `wolio.lock`, SHA-256 content-addressed `~/.wolio/cache/`, offline install | `wolio-semantic`, `wolio-resolver`, `wolio-diagnostics` |
| `crates/wolio-format` | `M07` (`T072`) | Comment-preserving 4-space LF canonical formatter with post-format `sym_` ID check | `wolio-lexer`, `wolio-parser`, `wolio-ast`, `wolio-semantic`, `wolio-diagnostics` |
| `crates/wolio-lsp` | `M07` (`T073`) | Incremental stdio LSP server with in-memory buffer isolation and safe workspace rename | `wolio-parser`, `wolio-semantic`, `wolio-verify`, `wolio-format`, `wolio-diagnostics` |
| `crates/wolio-cli` | `M01` (`T001`/`T004`) | CLI entrypoint composing active milestone crates and emitting human / `--json` envelopes | All active milestone service crates |

## 3. Hard architectural invariants

1. **No dependency cycles:** Cargo crate graph is a strict DAG.
2. **LLVM isolation:** `wolio-runtime`, `wolio-ir`, `wolio-semantic`, and
   `wolio-cli` (in M01–M04 builds) never depend on LLVM.
3. **Verified IR gate:** `VerifiedWolioIr` has private fields inside `wolio-ir`
   and can only be constructed by `wolio_ir::verify_ir(...)`.
4. **Single semantic model:** CLI, interpreter, IR lowering, agent protocol,
   formatter, and LSP all consume the same `wolio-semantic` graph.
