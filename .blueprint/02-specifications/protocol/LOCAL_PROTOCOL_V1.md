`.blueprint/02-specifications/protocol/LOCAL_PROTOCOL_V1.md`
# Local agent protocol V1

M04 provides local JSON-lines over stdin/stdout; no HTTP server, remote auth,
or network transport is included. The process runs under the invoking OS user.
Mutation payloads are defined by `MUTATION_SCHEMA_V1.md`; tier and lifecycle
rules are defined by `APPROVAL_MATRIX_V1.md`. This protocol constrains managed changes, not an agent with independent shell
access. Untrusted code execution needs an OS sandbox; metadata is not a sandbox.

Request envelope: `protocol_version: 1`, unique string `request_id`, `method`,
`base_snapshot`, and object `params`. Response repeats version/request_id and
contains exactly one of `result` or `error` (code, message, diagnostics).
Methods: inspect, impact, plan, apply, verify, finalize. Unknown versions/methods
fail before mutation. Queries require the snapshot they describe.

Plan params contain intent, operation array, allowed symbol/path scope, and
requested capabilities. Operations are create_declaration, replace_implementation,
replace_contract, rename_symbol, move_symbol, add_test, delete_symbol. Each carries
target ID (except create), expected old hash, and typed payload. Payloads encode
V1 AST nodes using tagged JSON objects; implementation bodies and predicates are
AST expressions, never raw source strings. String literal values and intent
strings remain ordinary data. The schema covers CORE_V1 productions only and
rejects unknown tags/fields, raw source, byte offsets, patches and arbitrary file
replacement. Parse/type checks still apply to emitted source. The mutation
engine renders canonical source from the validated AST and preserves unrelated
source/comments. It must not invent requires/ensures or widen effect/dependency
sets to make a plan pass. Missing lists retain CORE_V1's empty-set meaning.
Creation rejects an occupied semantic key; rename/move returns old/new identity
links and updates references as one scope-checked change. No high-level alias
such as create_operation bypasses these checks; create_declaration with kind
operation is the canonical V1 operation.

Plans return a plan ID, canonical AST diff hash, affected IDs, compatibility
classes, approval tier, required approvals, and diagnostics. Tier is computed
by the service from the complete diff and impact, never supplied by the agent:

- `AUTO`: implementation/test edits or private declaration creation/deletion,
  with no changed public interface, existing contract, error/effect/dependency
  sets, capabilities, policy, security, schema or architecture boundary.
- `POST_REVIEW`: changes to existing private contract syntax/intent only when
  signatures, error/effect/dependency sets and normalized predicates are
  identical, with no public contract impact. V1 cannot prove other predicate
  refinements. Complete technical evidence is required before publication.
- `HUMAN_REQUIRED`: all public interface/contract changes, other predicate
  changes (`unknown` included), existing error/effect/dependency set changes,
  package identity, capability/security, policy, schema and architecture-boundary
  changes. This is also the fallback for unclassified changes. V1 does not
  implement unsafe exceptions; approval cannot enable unsupported syntax.

The highest tier across the plan wins. New private operations with nonempty
error/effect/dependency sets require HUMAN_REQUIRED. Any affected definition
rename/move is at least POST_REVIEW, and HUMAN_REQUIRED when public or when
compatibility cannot be established. Tests may not authorize their own contract
changes; removing tests cannot discharge required evidence. AUTO/POST_REVIEW
never waive scope, checks or runtime guards. Deny overrides allow; default scope
and capabilities are deny. Scope matches normalized
project-relative paths and IDs; symlink escape and `..` traversal are rejected.

HUMAN_REQUIRED approval is a local owner-authored record outside agent-writable
scope: approver,
plan hash, base snapshot, capabilities, UTC expiry, and revocation state. It is
valid for one application only, is not delegable, and expires after 24 hours at
most. The project owner is the invoking repository maintainer; remote identity
federation is outside V1. Agent cannot write its own approval records.

Apply stages changes in an isolated temporary workspace, validates scope both
before and after, checks the current snapshot, runs verification, and acquires
an exclusive project mutation lock before compare-and-swap publication. Changed
base or approval returns WOLIO-STALE-001. A journal records old/new content hashes;
crash recovery completes or restores the entire managed change before another
mutation. Never discard unrelated user edits. External side effects are not
rolled back by source rollback. Tests receive only explicitly granted execution
capabilities. `verify` establishes technical evidence; `apply` publishes a
verified authorized change; `finalize` binds evidence to the published snapshot.
Finalization does not prove business intent or grant merge/release approval.
AUTO/POST_REVIEW may finalize without synchronous human interaction when all
required gates pass; not_run/fail or unknown required evidence blocks it.
Runtime-checked contracts are reported as such, never as universal proof.

POST_REVIEW publication appends a durable review record (plan/diff hash,
base/result snapshots, actor, affected IDs, evidence references, status pending)
in the same recoverable transaction. Owner review appends accepted/rejected
disposition tied to that record; agents cannot resolve their own review. Local
query output exposes pending reviews for PR audit; GitHub/remote integration is
not required in V1. Pending review does not block local execution/finalization.
Rejection requires a new repair/revert plan against the current snapshot, never
silent rollback over later work. Missing review record blocks POST_REVIEW
publication, and failed finalization never creates successful evidence.

Repeated request IDs with identical payload replay the recorded result; changed
payload with the same ID fails. Records are append-only JSONL under `.wolio-log/`
(explicit project artifacts, not rebuildable cache). Secret values are excluded.
Repair is another plan with the same constraints; default budget is 3 attempts,
then report unresolved diagnostics. No suppression or automatic contract weakening.

Acceptance: valid edit, stale edit, missing/expired/revoked approval, traversal,
symlink escape, denied scope, failed verification, crash recovery, duplicate
request, changed duplicate payload, and preservation of concurrent user edits.
Also require tier precedence, normalized-private POST_REVIEW, unknown/public
contract HUMAN_REQUIRED, raw-text rejection, no self-review, pending review
recovery/replay and rejection followed by a new repair plan.
