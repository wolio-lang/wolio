`.blueprint/GLOSSARY.md`
# Wolio Glossary (V1–V3)

- **Nominal type (`type Name = Repr;`):** A distinct semantic type constructed via `Name(x)` and unwrapped via `.value` that never implicitly coerces with its underlying representation or other nominal types.
- **Declaration identity (`sym_<sha256>`):** Deterministic SHA-256 identifier computed from `("wolio.symbol.v1", package_id, module, kind, name)`, independent of file paths, formatting, or byte offsets.
- **Total bounded loop (`for..in..with..bound..invariant`, V2):** An immutable accumulator loop guaranteed to terminate and checked against `bound` on entry and `invariant` on entry, each step, and exit.
- **Pure `predicate` (V2):** A total, effect-free, acyclic boolean declaration callable inside `requires`, `ensures`, `bound`, and `invariant` clauses.
- **Anti-workaround analyzer (`WOLIO-ANALYZER-001`..`005`):** Static compiler pass rejecting diagnostic suppression (`001`), swallowed errors returning dummy `Ok` (`002`), padded unused `dependencies`/`effects` (`003`), forbidden raw or constant-folded literals (`004`), and warning on duplicate arms or test-literal mirroring (`005`).
- **Verified IR (`VerifiedWolioIr`):** Opaque token constructed exclusively by the `wolio-ir` structural verifier and required by `wolio-codegen` before LLVM lowering.
- **Package identity:** Canonical repository path (`github.com/<owner>/<repo>`) bound in `wolio.lock` with commit, source, contract, and dependency SHA-256 hashes.
