`.blueprint/DOCUMENT_AUTHORITY.md`
# Document authority — revision 1.4

This register is the entry point for the entire Wolio blueprint. Revision 1.4 is
a pre-implementation freeze candidate across the three-generation roadmap
(`V1 -> V2 -> V3`), not owner acceptance or shipped software.

## 1. Zero-clutter authority rule

In Revision 1.4, all 46 superseded historical draft files (`LANGUAGE_SPEC.md`,
`SCOPE.md`, `WTP_SPEC.md`, historical crate sketches, and `M00-*` drafts) have
been removed from `.blueprint/`. Every file remaining in `.blueprint/` is active,
normative for its version, and non-contradictory.

## 2. Binding reading order

1. **Authority & Version Baselines:**
   - This register (`DOCUMENT_AUTHORITY.md`)
   - [BASELINE-V1.md](BASELINE-V1.md): V1 scope (`M01`–`M07`, `T001`–`T074`)
   - [BASELINE-V2.md](BASELINE-V2.md): V2 scope (`M08`–`M11`, `T201`–`T216`)
   - [BASELINE-V3.md](BASELINE-V3.md): V3 scope (`M12`–`M15`, `T301`–`T315`)
   - [00-product/PROJECT_COVERAGE.md](00-product/PROJECT_COVERAGE.md): complete V1/V2/V3 capability matrix.
2. **Normative Language, Protocol & Verification Specifications:**
   - V1 Language: [02-specifications/language/CORE_V1.md](02-specifications/language/CORE_V1.md)
   - V2 Language: [02-specifications/language/CORE_V2.md](02-specifications/language/CORE_V2.md)
   - V3 Language: [02-specifications/language/CORE_V3.md](02-specifications/language/CORE_V3.md)
   - Behavioral Contracts: [02-specifications/CONTRACT_DESIGN.md](02-specifications/CONTRACT_DESIGN.md)
   - Semantic Identity & Graph: [02-specifications/SEMANTIC_MODEL.md](02-specifications/SEMANTIC_MODEL.md)
   - Shared CLI, Manifest & Hash Contracts: [02-specifications/INTERFACE_CONTRACTS.md](02-specifications/INTERFACE_CONTRACTS.md)
   - Agent Protocol & Mutation Schema: [02-specifications/protocol/LOCAL_PROTOCOL_V1.md](02-specifications/protocol/LOCAL_PROTOCOL_V1.md), [02-specifications/protocol/MUTATION_SCHEMA_V1.md](02-specifications/protocol/MUTATION_SCHEMA_V1.md), [02-specifications/protocol/APPROVAL_MATRIX_V1.md](02-specifications/protocol/APPROVAL_MATRIX_V1.md)
   - Package Manager: [02-specifications/package-manager/PACKAGE_MANAGER_SPEC.md](02-specifications/package-manager/PACKAGE_MANAGER_SPEC.md)
   - Diagnostics & Evidence: [02-specifications/verification/DIAGNOSTIC_SPEC.md](02-specifications/verification/DIAGNOSTIC_SPEC.md), [02-specifications/verification/EVIDENCE_MODEL.md](02-specifications/verification/EVIDENCE_MODEL.md)
   - Version Boundaries: [02-specifications/EXTENSION_BOUNDARIES.md](02-specifications/EXTENSION_BOUNDARIES.md)
3. **Implementation, Quality & Delivery:**
   - [03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md](03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md)
   - [03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md](03-implementation/IMPLEMENTATION_SPECIFICATION_V2.md)
   - [03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md](03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md)
   - [03-implementation/DELIVERY_POLICY.md](03-implementation/DELIVERY_POLICY.md)
   - [05-quality/ACCEPTANCE_MATRIX.md](05-quality/ACCEPTANCE_MATRIX.md), [05-quality/DEFINITION_OF_DONE.md](05-quality/DEFINITION_OF_DONE.md), [05-quality/RELEASE_CRITERIA.md](05-quality/RELEASE_CRITERIA.md)
   - [06-project-management/DELIVERY_PLAN.md](06-project-management/DELIVERY_PLAN.md), [06-project-management/POST_V1_PLAN.md](06-project-management/POST_V1_PLAN.md), [06-project-management/AI_BUG_TAXONOMY.md](06-project-management/AI_BUG_TAXONOMY.md)
   - [04-tasks/EXECUTION_CONTRACTS.md](04-tasks/EXECUTION_CONTRACTS.md), [04-tasks/task-index.md](04-tasks/task-index.md)
4. **Self-Contained Task Contracts (`04-tasks/`):**
   - Every V1 task (`T001`–`T074`), every V2 task (`T201`–`T216`, also summarized in [04-tasks/V2-TASKS.md](04-tasks/V2-TASKS.md)), and every V3 task (`T301`–`T315`, also summarized in [04-tasks/V3-TASKS.md](04-tasks/V3-TASKS.md)) has its own standalone, self-contained task file embedding its required crate boundaries, rules, and diagnostic codes.

## 3. Revision 1.4 governance

Revision 1.4 incorporates [ADR-008](01-architecture/decisions/ADR-008-multi-version-anti-workaround-roadmap.md)
and [ADR-007](01-architecture/decisions/ADR-007-deterministic-identity-and-gated-automation.md).
Where two normative documents appear to disagree, stop the affected task, record
the exact conflict under `.execution/blockers/`, and resolve it by amendment; do
not choose whichever is convenient. Execution facts are authoritative only in
`.execution/`; document statuses never claim code exists. Owner baseline
acceptance is recorded in `.execution/baseline-acceptance.md` before `T001`.
