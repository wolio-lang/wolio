# Approval matrix V1

This document is normative with `LOCAL_PROTOCOL_V1.md` and anchored in
[ADR-010](../../01-architecture/decisions/ADR-010-vibe-native-zero-inspection-architecture.md)
(Pillar 2: Macro-Capability Envelopes). It defines deterministic risk-tier
selection, autonomous agent evolution boundaries, and publication/review lifecycle.

## 1. Architectural Philosophy: Macro-Capability vs Micro-AST

Under the Vibe-Native Zero-Code Inspection Architecture, human engineers do not inspect
micro-AST operation diffs, private function additions, internal error variants, or
routine internal refactorings. The human architect specifies High-Level Intent, Domain
Schemas, and Macro-Capability Envelopes (`[capabilities]` in `wolio.toml`).

Any internal codebase evolution that operates strictly within the pre-approved
macro-capability envelope is fully autonomous (`AUTO`). Human intervention is reserved
exclusively for external security boundary escalations, breaking public API contracts,
or computational resource budget violations.

## 2. Tier Precedence

`HUMAN_REQUIRED` outranks `POST_REVIEW`, which outranks `AUTO`. A plan's tier is
the highest tier of any operation or transitive impact across the entire plan.
The agent cannot request, lower, or override the calculated tier.

```text
       ┌────────────────────────────────────────────────────────┐
       │                     HUMAN_REQUIRED                     │
       │  - External capability escalation (net/fs/WASI)        │
       │  - Breaking public API contract changes                │
       │  - Runtime/computational resource budget violations    │
       └───────────────────────────▲────────────────────────────┘
                                   │ outranks
       ┌───────────────────────────┴────────────────────────────┐
       │                      POST_REVIEW                       │
       │  - Additive public declarations (non-breaking)         │
       │  - Permitted multi-module architecture reorganization  │
       │  (Asynchronous audit record in .wolio-log/; non-blocking)│
       └───────────────────────────▲────────────────────────────┘
                                   │ outranks
       ┌───────────────────────────┴────────────────────────────┐
       │                          AUTO                          │
       │  - All internal private declarations & implementations  │
       │  - Internal error sets & internal capability I/O        │
       │  - Private refactoring, renames, and module moves      │
       │  - Mechanical auto-pruning (unused deps/imports)       │
       │  - Test additions, property fuzzing & hermetic specs   │
       └────────────────────────────────────────────────────────┘
```

## 3. Tier Selection Matrix

| Change Category | Tier | Authoritative Rationale |
|---|---|---|
| **Internal implementation edit** | `AUTO` | Bounded technical edit with unchanged external interface. |
| **New private declaration** (operation, record, enum, type alias) | `AUTO` | Autonomous internal evolution within module boundary. |
| **New internal error variant** or private error set modification | `AUTO` | Internal error evolution within capability envelope; no public caller breakage. |
| **Internal effect / I/O declaration** within existing `[capabilities]` | `AUTO` | Pre-approved by project macro-capability envelope in `wolio.toml`. |
| **Private symbol rename or private module move** | `AUTO` | Internal refactoring; semantic identity and references updated atomically. |
| **Mechanical auto-pruning** (unused dependencies, unreferenced imports) | `AUTO` | Deterministic compiler AST rewrite pass; cleans bloat without human interruption. |
| **Test addition or test update** conforming to hermetic schemas | `AUTO` | Verifies technical obligations without altering production authority. |
| **Private contract refinement** | `AUTO` | Internal invariant checked mechanically by compiler & critic agent. |
| **Additive public declaration** (new `pub` operation/type, non-breaking) | `POST_REVIEW` | Non-breaking public boundary expansion; logged for asynchronous audit. |
| **Public module reorganization** (satisfying `[architecture]` policy) | `POST_REVIEW` | Provenance and module structure audit; does not break existing callers. |
| **External capability escalation** (new network domain, fs path outside sandbox) | `HUMAN_REQUIRED` | Boundary security change; requires explicit project owner authorization. |
| **Breaking public API schema change** (delete `pub`, change signature/type) | `HUMAN_REQUIRED` | Violates backward compatibility for external consumers. |
| **Public contract weakening** (weakening `ensures` or breaking invariants) | `HUMAN_REQUIRED` | Violates public caller assumptions and domain guarantees. |
| **Runtime / computational budget violation** (exceeding `timeout_ms`/`max_memory_mb`) | `HUMAN_REQUIRED` | Resource consumption envelope breach; requires quota grant. |
| **Verification bypass or unsafe injection** (`WOLIO-ANALYZER-001`) | `REJECT` | Zero-Unsafe Invariant violation; permanently rejected by compiler. |
| **Out-of-scope edit or traversal escape** (`WOLIO-SCOPE-001`) | `REJECT` | Path/symbol scope violation; rejected before staging. |

Deleting tests that are required evidence is rejected, regardless of tier.
Compiler warnings do not raise a tier; fatal analyzer errors reject the plan before
tier publication. A plan with failed, missing, `not_run`, or required `unknown`
evidence cannot be applied or finalized.

## 4. Operational Definition of HUMAN_REQUIRED Triggers

Human intervention is strictly gated and triggered **ONLY** under three explicit conditions:

### Trigger A: External Capability Escalation
Any mutation that attempts to expand the system's external interaction boundaries beyond
the pre-approved `[capabilities]` in `wolio.toml`, specifically:
1. Adding new external network hosts, domains, IP ranges, or protocols under `[capabilities.network]`.
2. Granting filesystem access outside the declared sandboxed roots under `[capabilities.filesystem]`.
3. Adding new foreign WASI component imports, hardware devices, or process execution privileges.

### Trigger B: Breaking Public API Schema Changes
Any mutation that breaks contractual backward compatibility for external consumers of
the module or package, specifically:
1. Deleting or un-exporting an existing `pub` declaration (operation, type, variant, field).
2. Modifying the arity, parameter types, or return type of an existing `pub` operation.
3. Removing or renaming an enum variant of an exported `pub enum`.
4. Removing fields from an exported public record or adding mandatory fields without default values.
5. Weakening postconditions (`ensures`) or public nominal refinement types on exported operations.

### Trigger C: Runtime and Computational Resource Budget Violations
Any mutation or configuration edit that breaches the execution budget envelopes configured
in `wolio.toml`, specifically:
1. Operations exceeding the maximum CPU execution time envelope (`timeout_ms`).
2. Workloads exceeding the resident memory allocation ceiling (`max_memory_mb`).
3. Workloads violating disk storage, mailbox queue depth, or thread/actor spawn quotas.

## 5. Lifecycle and Disposition

### AUTO Lifecycle
1. The mutation plan is staged and verified in an isolated workspace.
2. The Mechanical Auto-Prune pass clears any trivial bloat (e.g. unused imports or unused dependencies).
3. If all compiler type checks, refinement invariants, and test suites pass, `apply` commits the change via atomic Compare-And-Swap (CAS).
4. Evidence is bound via `finalize` without synchronous human interaction.

### POST_REVIEW Lifecycle
1. `POST_REVIEW` plans apply, run, test, and produce local build artifacts while review is pending.
2. Atomically with publication, the agent runtime records an append-only audit record in `.wolio-log/`:
   - Plan hash, diff hash, base/result snapshots, actor, affected IDs, evidence references, and timestamp.
3. Review status `pending` does NOT block local developer workflows or local testing.
4. However, `pending` or `rejected` review records strictly block production release packaging (`wolio release`) and official baseline acceptance.
5. The repository owner records disposition:
   - `pending -> accepted`: Clears release eligibility gating.
   - `pending -> rejected`: Requires a new repair or revert plan against the current snapshot; never silently rolls back downstream work.
6. AI agents cannot resolve their own review records (`accepted`/`rejected`).

### HUMAN_REQUIRED Lifecycle
1. When a plan touches a `HUMAN_REQUIRED` trigger, planning produces a cryptographic plan hash, diff hash, and explicit capability escalation request.
2. The agent cannot apply or finalize the plan until an authorized owner records a valid approval record.
3. The approval record must be placed outside agent-writable workspace scope (e.g., `.wolio/approvals/` protected by OS permissions).
4. An approval record binds strictly to:
   - Exact `plan_hash` and `diff_hash`.
   - Exact `base_snapshot` hash.
   - Requested capability delta or public schema diff.
   - Approver identity and cryptographic signature.
   - Expiration timestamp (maximum 24 hours UTC).
   - Single-use consumption state.
5. Once applied, the approval record is permanently consumed and marked used. Approval cannot legalize invalid syntax, failed verification, or a zero-unsafe policy violation.
