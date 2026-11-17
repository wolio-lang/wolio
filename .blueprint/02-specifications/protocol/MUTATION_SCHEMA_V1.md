# Structured mutation schema V1

This document is normative for M04. It is independent of any AI client,
editor, shell, or orchestration tool. The JSON-lines transport is defined by
`LOCAL_PROTOCOL_V1.md`; this document specifies the schemas for mutation plans,
operation payloads, the Context Spine Generator, and Mechanical Auto-Pruning,
anchored in [ADR-010](../../01-architecture/decisions/ADR-010-vibe-native-zero-inspection-architecture.md).

## 1. Canonical Encoding and Hashing

Requests and responses use UTF-8 JSON. Object keys are sorted lexicographically
for hashing. Unknown object keys are rejected. Arrays preserve order unless a
field says it is a set; set arrays are sorted by their canonical JSON value.
Integers are signed decimal JSON integers in the CORE_V1 Int range. No floats,
null values (except create preconditions), duplicate keys, comments, JSON patches,
byte offsets, arbitrary line diffs, or arbitrary file paths are accepted.

To optimize for natural LLM construction and eliminate the token and nesting overhead
of manual JSON-AST serialization, implementation bodies, contract clauses, and
declarations may be provided either as typed AST objects OR as canonical Wolio
source strings (`code: string`). When `code` is supplied, `wolio-agent` parses it
server-side via `wolio-parser` into a verified AST before diffing; raw file-level
writes and out-of-scope edits remain strictly forbidden.

The canonical mutation plan hash is SHA-256 over:

```text
length_prefix("wolio.mutation.v1") || length_prefix(base_snapshot)
  || length_prefix(canonical_json(params))
```

Length prefixes follow `INTERFACE_CONTRACTS.md`. Scope and capabilities are included.
The plan hash is independent of transport request ID, timestamps, actor names,
and diagnostic wording.

## 2. Plan Request Envelope

```json
{
  "intent": "implement order cancellation with inventory replenishment",
  "operations": [],
  "scope": {
    "paths": ["src/orders.wol"],
    "symbols": ["sym_order_cancel"]
  },
  "capabilities": ["network.http_client"]
}
```

- `intent`: Non-empty human-readable specification of purpose (provenance metadata).
- `operations`: Non-empty array of structured operations (max 256 per plan).
- `scope`: Non-empty object containing `paths` and `symbols` arrays. Empty scope matches nothing.
- `capabilities`: Array of macro-capability identifiers requested for this operation
  (validated against `wolio.toml` `[capabilities]`).

## 3. Plan Response Envelope

The response to a successful `plan` invocation provides full semantic diff metadata,
derived approval tier, MCTS branching checkpoints, and Mechanical Auto-Prune actions:

```json
{
  "plan_id": "plan_01j7...",
  "plan_hash": "sha256...",
  "ast_diff_hash": "sha256...",
  "affected_ids": ["sym_order_cancel", "sym_inventory_restore"],
  "approval_tier": "AUTO",
  "required_approvals": [],
  "auto_pruned": [
    {
      "kind": "unused_dependency",
      "target": "sym_order_cancel",
      "symbol": "sym_unused_helper",
      "reason": "Declared in dependencies but never invoked in implementation"
    }
  ],
  "branch_info": {
    "branch_id": "branch_mcts_04",
    "parent_snapshot": "snap_sha256_base...",
    "rollback_checkpoint": "chk_sha256_valid..."
  },
  "diagnostics": []
}
```

- `approval_tier`: Exactly one of `"AUTO"`, `"POST_REVIEW"`, or `"HUMAN_REQUIRED"`. Derived
  deterministically by the service per `APPROVAL_MATRIX_V1.md`.
- `auto_pruned`: Array of non-fatal entity descriptors pruned automatically by the compiler's
  Mechanical Auto-Prune pass.
- `branch_info`: Metadata anchoring the plan in the MCTS exploration tree for rollback on failure.

## 4. Operation Envelope

Every operation has exactly `kind`, `target`, `expected_old_hash`, and `payload`:

```text
create_declaration
replace_implementation
replace_contract
rename_symbol
move_symbol
add_test
delete_symbol
```

- `target`: Omitted only for `create_declaration`. Semantic symbol ID (`sym_...`).
- `expected_old_hash`: `null` only for `create_declaration`. Otherwise, SHA-256 of the target AST.
- `payload`: Typed AST object or `{code: string}` snippet matching the operation kind.

The service rejects conflicting operations, unknown kinds, and any operation whose target
falls outside the declared scope.

## 5. Typed AST Values

Expressions are tagged objects with exactly one `kind`:
```text
int       {kind: "int", value: number}
string    {kind: "string", value: string}
bool      {kind: "bool", value: boolean}
name      {kind: "name", path: [identifier, ...]}
unary     {kind: "unary", op: "!"|"-", value: expr}
binary    {kind: "binary", op: string, left: expr, right: expr}
call      {kind: "call", path: [identifier, ...], args: [expr, ...]}
record    {kind: "record", path: [identifier, ...], fields: [{name, value}, ...]}
if        {kind: "if", condition: expr, then: expr, else: expr}
match     {kind: "match", subject: expr, arms: [{pattern, value}, ...]}
field     {kind: "field", value: expr, name: identifier}
unit      {kind: "unit"}
block     {kind: "block", lets: [{name, value}, ...], value: expr}
```

Patterns:
```text
constructor {kind: "constructor", path: [identifier, ...], binding?: identifier}
bool        {kind: "bool", value: boolean}
```

Types:
```text
name        {kind: "name", path: [identifier, ...]}
application {kind: "application", path: [identifier, ...], args: [type, ...]}
refinement  {kind: "refinement", base: type, predicate: [identifier, ...]}
```

## 6. Complete Declaration Shapes

```text
create payload = {module: identifier, declaration: declaration_shape | {code: string}}
nominal declaration = {kind: "nominal", name: identifier, public: boolean, type: type}
record declaration = {kind: "record", name: identifier, public: boolean, fields: [{name, type}]}
enum declaration = {kind: "enum", name: identifier, public: boolean, variants: [{name, type?}]}
operation declaration = {kind: "operation", name: identifier, public: boolean,
                         parameters: [{name, type}], returns: type, contract: contract}
implementation declaration = {kind: "implementation", name: identifier, body: block}
test declaration = {kind: "test", name: identifier, operation: path, body: block}
contract = {requires: [expr], ensures: [expr], effects: [path],
            dependencies: [path], errors: [path], intent?: string}
replace_implementation payload = {body: block} | {code: string}
replace_contract payload = {contract} | {code: string}
rename_symbol payload = {new_name: identifier}
move_symbol payload = {new_module: identifier}
add_test payload = {module: identifier, name: identifier, body: block} | {module: identifier, name: identifier, code: string}
delete_symbol payload = {}
```

When `{code: string}` is provided, `wolio-agent` parses the string into the exact AST shape
above using `wolio-parser`. If parsing fails, validation immediately returns `WOLIO-SYNTAX-001`
without staging. Valid ASTs are diffed against `expected_old_hash`.

## 7. Context Spine Specification (`.wolio/context-spine.json`)

The Context Spine is a compact, high-density JSON index generated by the compiler to give
multi-session agents global architectural visibility within a ≤ 2,000 token budget:

### File Location
`.wolio/context-spine.json` (also queryable via `method: "inspect", params: {"kind": "context_spine"}`).

### JSON Schema
```json
{
  "spine_version": 1,
  "project_name": "ecommerce_core",
  "project_snapshot": "snap_sha256...",
  "macro_capabilities": {
    "network": ["api.payment.internal"],
    "filesystem": ["data/export"],
    "resources": {
      "timeout_ms": 5000,
      "max_memory_mb": 512
    }
  },
  "modules": [
    {
      "name": "orders",
      "path": "src/orders.wol",
      "types": [
        {
          "name": "OrderId",
          "kind": "nominal",
          "public": true,
          "base_type": "Int",
          "refinement": "PositiveInt"
        },
        {
          "name": "OrderStatus",
          "kind": "enum",
          "public": true,
          "variants": ["Pending", "Paid", "Cancelled"]
        }
      ],
      "operations": [
        {
          "name": "cancel_order",
          "public": true,
          "parameters": [{"name": "id", "type": "OrderId"}],
          "returns": "Result<Unit, OrderError>",
          "effects": ["Console::Write"],
          "dependencies": ["inventory::replenish"]
        }
      ],
      "dependencies": ["inventory", "accounts"]
    }
  ],
  "architecture_rules": {
    "layer": "domain",
    "forbidden_imports": ["infrastructure", "cli"]
  }
}
```

### Multi-Session Constraint
Agents are required to read `.wolio/context-spine.json` upon initialization of any session.
The compiler enforces:
1. **Zero Duplicate Types:** Declaring a type whose normalized structure duplicates an
   existing type in the Context Spine triggers `WOLIO-TYPE-002` (Duplicate Domain Type).
2. **Zero Orphan Scripts:** Every file must belong to a module listed in `modules`. Unmapped
   files trigger `WOLIO-MODULE-001`.

## 8. Mechanical Auto-Prune Schema

When the compiler detects trivial bloat during plan evaluation, it rewrites the AST diff
and populates the `auto_pruned` response field:

```json
{
  "kind": "unused_dependency" | "unused_import" | "dead_binding",
  "target": "sym_...",
  "symbol": "sym_...",
  "reason": "string description of deterministic prune"
}
```

Auto-pruning is deterministic and idempotent. It eliminates compilation failure on
routine cleanups and prevents agent retry thrashing.

## 9. Rejection Contract

Plans are rejected before staging with structured diagnostics on:
1. Malformed JSON, uncanonical key ordering, or missing required fields (`WOLIO-SYNTAX-001`).
2. Targets or paths outside declared scope, `..` path traversals, symlink escapes (`WOLIO-SCOPE-001`).
3. Out-of-bounds AST depth (> 128) or operations count (> 256).
4. Attempted escalation of external capabilities or breaking public API schema without
   valid `HUMAN_REQUIRED` owner approval tokens.
5. In-source suppression directives or zero-unsafe invariant violations (`WOLIO-ANALYZER-001`).
