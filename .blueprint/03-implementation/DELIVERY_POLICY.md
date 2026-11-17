`.blueprint/03-implementation/DELIVERY_POLICY.md`
# Delivery, toolchain, and cross-version bug-fix policy

## 1. Platform and toolchain pinning

Rust/Cargo implements the compiler; Linux x86_64 is the V1–V3 supported platform.
Other platforms are best-effort until added with CI. `T001` selects the
then-current stable Rust release, records its exact version in
`rust-toolchain.toml`, commits `Cargo.lock`, and uses GitHub Actions on a pinned
runner image. No floating version is permitted in recorded release evidence.
`M05` similarly pins a supported LLVM major and exact build/container digest
before backend implementation.

## 2. Interpreter-first and verified native lowering

Typed semantic expressions execute sequentially in `wolio-runtime` with checked
arithmetic and contract guards. A host Rust panic is an internal compiler failure
(exit code `3`), never a language `Result`.

`M05` lowers the same checked semantics into typed SSA-like IR with explicit
blocks, branches, calls, constructors, tags, arithmetic guards, and contract
guards. The IR verifier rejects ill-typed edges and missing terminators. Only the
verifier may construct the opaque `VerifiedWolioIr` input accepted by codegen. It
binds immutable IR to the snapshot and completed static checks; transformed IR
must be reverified. Runtime guards remain mandatory in release builds. `run` and
`test` use the interpreter by default; `build` explicitly invokes native codegen.
`wolio-codegen` emits verified textual LLVM IR (`.ll`) and compiles to native
binaries via the pinned `clang` toolchain (linking with `libwolio_runtime.a`),
ensuring hermetic reproducibility without Cargo `llvm-sys` FFI build failures.
LLVM optimization must preserve faults and guards; differential fixtures
comparing interpreter and native execution are the release gate. Immutable heap
values use reference counting initially (with copy-on-write optimization for
`List`/`Map` in V3 `M12`); no pointer identity is exposed.

## 3. Cross-version Bug-Fix & Regression Gate Policy (`V1 -> V2 -> V3`)

To prevent technical debt and regressions as Wolio evolves from V1 (`CORE_V1.md`)
to V2 (`CORE_V2.md`) and V3 (`CORE_V3.md`):

1. **Dedicated Inter-Version Stabilization Milestones:**
   - `M08` (`T201`–`T203`) is a mandatory **V1 Bug-Fix & Regression Gate** before
     any V2 language feature (`M09`) can be implemented.
   - `M12` (`T301`–`T303`) is a mandatory **V2 Bug-Fix, Memory & Performance Gate**
     before any V3 standard library or stateful feature (`M13`) can be implemented.
2. **Regression-First Bug Protocol (`BUG-nnnn`):**
   - Every compiler, runtime, protocol, or tooling defect discovered during
     milestone execution or benchmark campaigns (`M03`, `M06`, `M11`, `M15`) is
     recorded in `.execution/reports/BUG-nnnn.md`.
   - Every bug fix MUST first add a minimal failing conformance fixture under
     `tests/conformance/regressions/BUG-nnnn/` that reproduces the defect, then
     fix the implementation without weakening any existing contract or acceptance
     case.
   - All `tests/conformance/regressions/**` fixtures run in CI on every subsequent
     task across V1, V2, and V3.
3. **Strict Upward Version Isolation:**
   - Upgrading the compiler binary to V2 or V3 must never change the behavior,
     diagnostics, or `sym_` hashes of a project declaring `[language] version = "1"`
     in `wolio.toml`.

## 4. Manifest, versions, and M07 services

Use `[package] name/version`, `[language] version = "1" | "2" | "3"`, and
`[entrypoint] module/operation`. `[module] path` is required only for published
Git packages. Compiler, language, diagnostic schema, protocol, and package
versions are independent. Unsupported major versions fail explicitly
(`WOLIO-FEATURE-001`).

Release artifacts: Linux x86_64 `.tar.gz`, SHA-256 checksums, exact toolchain/lock
metadata, license, and release notes. `install --locked --offline` never resolves
new versions or accesses the network.

Formatter (`wolio fmt`) parses and prints canonical syntax, preserves `//`
comments, uses four-space indentation and LF, is idempotent, and verifies that
all `sym_` declaration IDs and clause identities are unchanged after formatting.
LSP (`wolio lsp`) supports `initialize`/`shutdown`, incremental document updates,
diagnostics, hover, definition, references, and version-checked workspace rename
using the shared semantic snapshot model without writing unsaved buffers to disk.
