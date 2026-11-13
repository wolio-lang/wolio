`.blueprint/MAP.md`
# Blueprint map (Revision 1.4 — V1, V2, and V3)

Revision 1.4 freeze candidate; zero conflicting historical documents remain.

## 1. Master baselines and authority

1. [Document authority](DOCUMENT_AUTHORITY.md)
2. [V1 baseline (`M01`–`M07`)](BASELINE-V1.md)
3. [V2 baseline (`M08`–`M11`: loops, collections, Decimal, predicates, architecture)](BASELINE-V2.md)
4. [V3 baseline (`M12`–`M15`: `std::*`, capability I/O, secrets, transactions, fuzzing)](BASELINE-V3.md)
5. [Whole-project coverage](00-product/PROJECT_COVERAGE.md)
6. [Delivery plan](06-project-management/DELIVERY_PLAN.md)
7. [Task index (`T001`–`T074`, `T201`–`T216`, `T301`–`T315`)](04-tasks/task-index.md)
8. [Acceptance matrix](05-quality/ACCEPTANCE_MATRIX.md)
9. [Ubuntu and agent quickstart](07-ai-context/GETTING_STARTED.md)

## 2. Self-contained task lookup

Every V1 task file (`04-tasks/Tnnn-*.md`) and V2/V3 task specification
([V2-TASKS.md](04-tasks/V2-TASKS.md), [V3-TASKS.md](04-tasks/V3-TASKS.md))
embeds its required crate boundaries, rules, and diagnostic codes. Only consult
the normative reference below if an implementation detail needs deeper context:

| Area | Normative specification |
|---|---|
| V1 Lexer, parser, types, runtime | [02-specifications/language/CORE_V1.md](02-specifications/language/CORE_V1.md) + [02-specifications/INTERFACE_CONTRACTS.md](02-specifications/INTERFACE_CONTRACTS.md) |
| V2 Bounded loops, collections, `Decimal`, `predicate`, `[architecture]` | [02-specifications/language/CORE_V2.md](02-specifications/language/CORE_V2.md) + [BASELINE-V2.md](BASELINE-V2.md) |
| V3 Standard library `std::*`, `Secret<T>`, mocks, `transaction`, `property` fuzzing | [02-specifications/language/CORE_V3.md](02-specifications/language/CORE_V3.md) + [BASELINE-V3.md](BASELINE-V3.md) |
| Contracts, errors, effects, dependencies | [02-specifications/CONTRACT_DESIGN.md](02-specifications/CONTRACT_DESIGN.md) |
| Graph, `sym_` identity, impact, provenance | [02-specifications/SEMANTIC_MODEL.md](02-specifications/SEMANTIC_MODEL.md) |
| Diagnostics, analyzer `001`–`005`, evidence | [02-specifications/verification/DIAGNOSTIC_SPEC.md](02-specifications/verification/DIAGNOSTIC_SPEC.md) + [02-specifications/verification/EVIDENCE_MODEL.md](02-specifications/verification/EVIDENCE_MODEL.md) |
| Local agent protocol, AST mutations, risk tiers | [02-specifications/protocol/LOCAL_PROTOCOL_V1.md](02-specifications/protocol/LOCAL_PROTOCOL_V1.md) + [MUTATION_SCHEMA_V1.md](02-specifications/protocol/MUTATION_SCHEMA_V1.md) + [APPROVAL_MATRIX_V1.md](02-specifications/protocol/APPROVAL_MATRIX_V1.md) |
| Git packages, lockfile, SHA-256 cache | [02-specifications/package-manager/PACKAGE_MANAGER_SPEC.md](02-specifications/package-manager/PACKAGE_MANAGER_SPEC.md) |
| Crates, IR/LLVM, formatter, LSP, bug-fix gates | [03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md](03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md) + [03-implementation/DELIVERY_POLICY.md](03-implementation/DELIVERY_POLICY.md) |

Actual progress lives only in `.execution/current-state.md` and `.execution/task-status.md`.
