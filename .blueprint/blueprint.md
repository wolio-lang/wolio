# Wolio — Industrial AI-Native Programming Language Master Overview (V1–V3)

> Normative execution baselines are [BASELINE-V1.md](BASELINE-V1.md),
> [BASELINE-V2.md](BASELINE-V2.md), and [BASELINE-V3.md](BASELINE-V3.md). System
> architecture and decisions are governed by [DOCUMENT_AUTHORITY.md](DOCUMENT_AUTHORITY.md)
> and [ADR-010](01-architecture/decisions/ADR-010-vibe-native-zero-inspection-architecture.md).

## 1. Core thesis: Industrial Vibe Coding and Eliminating AI "Akal-Akalan"

Wolio is a purpose-built programming language designed from first principles for
**Industrial Vibe Coding (Zero-Code Inspection)**—a software engineering paradigm where:
1. **The Human is the High-Level Specifier & Architect:** The human defines domain schemas,
   high-level `intent` specifications, macro-capability envelopes, and integration policies.
   The human **never reads generated implementation code**, never writes low-level boolean
   proofs, and never inspects micro-AST mutation diffs.
2. **The Machine is the Autonomous Coder and Adversarial Verifier:** Implementation is
   generated, mutated, and maintained by autonomous AI agents under rigorous compiler mechanics.
   Verification is conducted via **Adversarial Dual-Agent Property Synthesis** (Generator vs.
   Critic Agent), backed by deterministic compiler gates. Human readability is explicitly
   subordinated to 100% deterministic, machine-verifiable safety.
3. **Multi-Session AI Pathologies are Eradicated at Compiler Level:**
   In large projects (50,000+ LOC), AI agents in later sessions suffer from context window
   amnesia. Missing global architectural context, conventional AI models resort to deceptive
   workarounds ("script akal-akalan"): creating loose monkey patches, orphan utility scripts
   (`fix_temp.py`), duplicate helper functions, error swallowing, and fake mock tables.
   Wolio structurally eradicates these failure modes:
   - **Semantic Context Digest (`.wolio/context-spine.json`):** The compiler maintains a
     compact (~2,000-token) digest of all global nominal types, refinement bounds, module
     dependencies, and actor state machines. Even in Session 10+, an agent ingests this spine
     to attain 100% global architectural awareness without reading function bodies, preventing
     type duplication and orphan logic.
   - **Zero-Tolerance Anti-Orphan Architecture:** Every line of code must bind to a Nominal
     Domain Module registered in `wolio.toml`. Unmapped scripts or orphan functions are
     instantly rejected at Tier 0 syntax/module resolution (`WOLIO-MODULE-001`).
   - **Automated Global Regression Gate:** Local mutations trigger whole-project verification.
     If a local fix breaks a distant module, the compiler emits a **Semantic Root-Cause Diagnostic**
     forcing the agent to fix the origin contract rather than patching downstream.
   - **Nominal Refinement Types:** Closed algebraic enums and nominal refinements prevent
     fake string sentinels (`status = "DONE_HACK"`) or synthetic mock bypasses.

## 2. Immutable architectural directives

Wolio enforces four non-negotiable architectural directives aligned with [ADR-010](01-architecture/decisions/ADR-010-vibe-native-zero-inspection-architecture.md):

1. **Zero-Unsafe Policy & Sandboxed WASI Component Model:**
   - Wolio has **NO `unsafe` keyword**, no raw pointers, no unchecked memory
     dereferencing, and no manual allocation/deallocation.
   - All memory is 100% safe by construction (immutable value semantics, CoW
     reference counting, and linear resource tokens).
   - Foreign library interop (V3) standardizes on the **WebAssembly (WASI) Component Model**
     using Wasm Interface Types (WIT). External C/Rust/Go libraries execute in capability-sandboxed
     linear memory with zero host address space exposure.
2. **Machine Leverage via Macro-Capabilities & Context Spine:**
   - Humans set coarse **Macro-Capability Envelopes** in `wolio.toml` (allowed network domains,
     filesystem roots, compute/memory quotas). Micro-AST diffs are abolished; all internal
     refactoring within the envelope is marked `AUTO`.
   - `HUMAN_REQUIRED` is triggered strictly for external capability escalation, breaking public
     schema changes, or quota violations.
   - Multi-attempt deadlocks are replaced by an **MCTS Search-Tree & Deterministic Auto-Fix**
     model with state rollbacks, where the compiler mechanically prunes trivial unused bloat.
3. **Hyper-Aggressive 6-Tier Diagnostic Engine with Root-Cause Attribution:**
   - Multi-tiered compiler complaints: Tier 0 (`FATAL_SYNTAX` & `ORPHAN_MODULE`), Tier 1 (`TYPE_RESOLUTION`),
     Tier 2 (`BEHAVIORAL_CONTRACT`), Tier 3 (`SECURITY_CAPABILITY`), Tier 4 (`ANTI_HACKING_ANALYZER`),
     and Tier 5 (`HYPER_STRICT_LINT`).
   - **Zero In-Source Suppression:** Directives like `#[allow(...)]` or `// nolint`
     do not exist. Any attempt to suppress a compiler warning or lint is a fatal error
     (`WOLIO-ANALYZER-001`).
   - Trivial errors (e.g. unused imports/dependencies) undergo a deterministic **Mechanical Auto-Prune Pass**,
     while behavioral violations produce Semantic Root-Cause Diagnostics.
4. **Hermetic Schema-Driven Fuzzing & Ephemeral Integration Sandboxes:**
   - Arbitrary, hallucinated manual mock tables (`with_mock`) are prohibited.
   - Unit tests use **Schema-Driven Fuzzing** synthesized directly from authoritative schemas
     (OpenAPI, JSON Schema, Protobuf).
   - Integration tests execute against **Ephemeral Container Live Sandboxes** (real PostgreSQL,
     Redis, and HTTP services via Testcontainers/Docker) managed deterministically by the harness.

## 3. Three-generation conflict-free architecture (`V1 -> V2 -> V3`)

```text
V1 (M01–M07: Immutable Nominal Core, Context Spine & Autonomous Agent Protocol)
  - Acyclic immutable core, nominal types, exhaustive Result/Option, first-class refinement types
  - Adversarial Dual-Agent Property Synthesis (Generator vs. Critic) replacing manual boolean contracts
  - Semantic Context Digest (.wolio/context-spine.json ~2,000 tokens) for multi-session global sync
  - Zero-tolerance anti-orphan module architecture (Tier 0 rejection for unregistered scripts)
  - Macro-Capability Envelopes in wolio.toml (AUTO internal evolutions; HUMAN_REQUIRED on boundary escalation)
  - MCTS search-tree with branch rollback and mechanical compiler auto-pruning for trivial bloat
  - VerifiedWolioIr -> textual LLVM IR (.ll) lowering + clang native parity, Git package manager, fmt, LSP
  - Forward-reserves all V2 and V3 keywords from T002 (including stream, actor, spawn, send, wit, foreign)
        │
        ▼
V2 (M08–M11: V1 Bug-Fix Gate + Structural Vectorized Streams & Relational Collections)
  - M08: Mandatory V1 Bug-Fix & Regression Gate (BUG-nnnn conformance suite)
  - Structural Termination & Vectorized Streams: map, filter, fold, chunk, paginate, zip, take
    (replaces error-prone manual loop bound/invariant proving; termination-by-construction)
  - Immutable List<T>, Map<K,V>, Set<T>, exact Decimal with explicit RoundingMode, UInt, Bytes, format(...)
  - Checked bitwise operations (bit_and, bit_or, bit_xor, bit_shl, bit_shr, bit_not)
  - Reusable pure acyclic predicate declarations callable in refinement types
  - [architecture] allow/deny module dependency rules + multi-module Agent Protocol v2
  - Full 24-task algorithmic comparative benchmark (M11 / T215)
        │
        ▼
V3 (M12–M15: V2 Bug-Fix Gate + WASI Component FFI, Actor Concurrency & Ephemeral Sandboxes)
  - M12: Mandatory V2 Bug-Fix & Copy-on-Write Performance Gate
  - Sandboxed WASI Component Model FFI (Wasm Interface Types - WIT; capability-sandboxed native modules)
  - Actor Model & Supervision Trees (process-isolated state, typed message mailboxes, fail-stop recovery trees)
  - Schema-Driven Property Fuzzing (OpenAPI/Protobuf schema synthesis; replaces manual with_mock)
  - Ephemeral Container Live Sandboxes (Testcontainers/Docker for real database and HTTP integration)
  - Fixed-width integer types (UInt8..UInt32, Int8..Int32) with checked arithmetic
  - Opaque Secret<T> redaction & validation provenance
  - Linear non-aliasing resource handles with transaction frames and atomic rollback
```
