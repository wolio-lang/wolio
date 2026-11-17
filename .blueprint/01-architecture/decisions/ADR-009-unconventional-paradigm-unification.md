# ADR-009 — Unconventional paradigm: deterministic dataflow, relational value collections, and contractual refinement

**Status:** Proposed for owner acceptance with blueprint revision 1.5  
**Date:** 2026-11-16

## Context

Conventional programming languages (C++, Java, Go, Rust, TypeScript, Python) were designed around human developers writing manual code and handling low-level machine or runtime idioms. When applied to **Industrial Vibe Coding**—where human architects specify contracts and AI models generate/mutate implementations—conventional paradigms create structural failure vectors:

1. **Shared-Memory & Function-Coloring Concurrency:**
   - Multi-threading with locks/mutexes creates data races, deadlocks, and non-reproducible heisenbugs.
   - `async/await` introduces "function coloring", pollutes call stacks, obscures error propagation, and invites AI error swallowing.
2. **Mutable Heap & Pointer-Based Graphs:**
   - Pointer-based data structures (trees, cyclic graphs, linked lists) create pointer aliasing, iterator invalidation, and cyclic memory leaks.
   - Mutable hash tables have non-deterministic iteration orders dependent on random hash seeds, producing flaky tests and non-reproducible AI code repairs.
3. **Inheritance & Complex Trait Hierarchies:**
   - Class inheritance introduces the fragile base class problem and implicit runtime overrides.
   - Complex trait/typeclass systems (orphan rules, higher-kinded types, trait explosion) generate multi-page compiler error cascades that trap AI agents in hallucinated repair loops.
4. **Ambient I/O & In-Place State Mutation:**
   - Unrestricted mutable variables (`let mut`) combined with ambient I/O allow partial state corruption when errors occur mid-operation.
   - Unit tests calling live external I/O become flaky and incentivize AI to write hardcoded mock bypasses.

Wolio explicitly rejects copying conventional language idioms. This ADR codifies Wolio's radical, first-principles paradigm overhaul.

## Decision

Wolio establishes four foundational architectural pillars that replace conventional programming models across V1, V2, and V3:

### 1. Zero-Coloring Deterministic Dataflow Parallelism (Replacing Threads & Async/Await)
- **Elimination of Concurrency Keywords:** Wolio has no `async`, `await`, `thread`, `fork`, `mutex`, `channel`, or `lock` keywords.
- **Compiler-Managed Dataflow DAG:** Pure operations (`effects []`) declaring static dependencies (`dependencies [...]`) form a deterministic Directed Acyclic Graph. The compiler and runtime scheduler automatically execute independent operations in parallel.
- **Strictly Deterministic Concurrency:** Given identical inputs and state snapshots, execution traces and outputs are mathematically identical regardless of core allocation (1 core vs. 128 cores). Data races are impossible by construction.

### 2. Canonical Relational-Algebraic Value Collections (Replacing Mutable Pointer Heaps)
- **Zero Raw Pointers & Immutability:** Pointer-linked node structures and mutable in-place hash updates are permanently rejected.
- **Canonical Content-Addressed Sets & Maps:** `Set<T>` and `Map<K, V>` are pure, immutable value spaces with deterministic total ordering. Two collection instances with identical elements are guaranteed to share identical canonical bitwise layouts and cryptographic content hashes, independent of insertion order.
- **Relational Identity Graph:** Graphs, trees, and entity relationships are modeled as relational tuples over deterministic identity tokens (`sym_id` / `NodeId`), traversed exclusively through bounded accumulator loops (`for ... in ... with ... { bound ...; invariant ...; }`). Unbounded recursion and pointer cycles cannot compile.

### 3. Contractual Refinement & Closed Sums (Replacing Inheritance & Trait Soup)
- **Zero Class Inheritance & Zero Open Trait Hierarchies:** Wolio rejects subtype inheritance and open trait polymorphism.
- **Nominal Branding + Refinement Predicates:** Types are specialized via nominal wrappers (`type ValidPort = nominal Int;`). Behavioral constraints are expressed as pure mathematical propositions via `predicate` declarations and enforced at function boundaries via `requires` and `ensures`.
- **Exhaustive Closed Sums:** Polymorphic heterogeneity is handled strictly through tagged `enum` definitions. All consumer expressions must exhaustively handle all variants (`match`), ensuring instant compile-time failure upon variant additions.

### 4. Linear Resource Frames & Snapshot Rollbacks (Replacing Pointer Mutability & Ambient I/O)
- **Linear Resource Handles:** External stateful resources (database handles, files, network sockets) are linear, non-aliasing tokens (`resource Handle`).
- **Explicit Frame Modification:** Operations altering resource state must declare `modifies [resource]`, define pre-state/post-state relations using `old(expr)`, and ensure invariant preservation across all paths.
- **Hardware/Runtime Atomic Rollback:** Any execution path yielding `Result::Err` or violating an invariant triggers an immediate automatic frame rollback, preventing partial state corruption.
- **Mandatory Hermetic Test Isolation:** Live I/O is strictly forbidden in `wolio test` and `wolio verify` (`WOLIO-EFFECT-002`). All I/O must be mocked via deterministic `with_mock` tables.

## Consequences

- **Radical Paradigm Integrity:** Wolio avoids the historical baggage, technical debt, and semantic compromises of conventional languages.
- **AI-Native Mechanical Leverage:** By eliminating data races, cyclic pointer leaks, function coloring, and trait cascades, AI agents work against clean, local, mathematically unambiguous compiler constraints.
- **Determinism Across the Toolchain:** Builds, tests, and execution traces are reproducible down to the exact bit representation across distributed worker environments.
- **Compatibility:** This decision aligns with and reinforces [ADR-007](ADR-007-deterministic-identity-and-gated-automation.md) and [ADR-008](ADR-008-multi-version-anti-workaround-roadmap.md), serving as the immutable philosophical framework for V1, V2, and V3.

