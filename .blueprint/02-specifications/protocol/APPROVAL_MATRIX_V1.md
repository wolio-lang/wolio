# Approval matrix V1

This document is normative with `LOCAL_PROTOCOL_V1.md`. It defines deterministic
tier selection and publication/review lifecycle.

## Tier precedence

`HUMAN_REQUIRED` outranks `POST_REVIEW`, which outranks `AUTO`. A plan's tier is
the highest tier of any operation or transitive impact. The agent cannot request,
lower, or override the tier.

| Change | Tier | Reason |
|---|---|---|
| implementation body with unchanged semantic sets | AUTO | bounded technical edit |
| test addition with unchanged production obligations | AUTO | no authority change |
| private declaration with empty errors/effects/dependencies | AUTO | no external obligation |
| private rename or module move | POST_REVIEW | identity/provenance impact |
| private contract whose normalized predicates/signature/sets remain identical | POST_REVIEW | evidence/audit impact |
| public rename, move, declaration, or deletion | HUMAN_REQUIRED | public API impact |
| any public contract change | HUMAN_REQUIRED | caller obligation impact |
| any non-identical predicate change | HUMAN_REQUIRED | V1 compatibility is unknown |
| error, effect, or dependency set change | HUMAN_REQUIRED | propagated obligation impact |
| nonempty error/effect/dependency on new private operation | HUMAN_REQUIRED | new obligation |
| capability, security, policy, schema, package identity, architecture change | HUMAN_REQUIRED | boundary change |
| unclassified or analyzer hard error | HUMAN_REQUIRED/reject | deny by default |

Deleting tests that are required evidence is rejected, regardless of tier.
Analyzer warnings do not raise a tier; analyzer errors reject the plan before
tier publication. A plan with failed, missing, `not_run`, or required `unknown`
evidence cannot be applied or finalized.

## Lifecycle

`AUTO` and approved `HUMAN_REQUIRED` plans may apply after verification and
finalize evidence. `POST_REVIEW` plans may apply, run, test, and produce local
build artifacts while review is pending, but cannot produce a release artifact
or become a recorded project baseline until accepted.

Review records are append-only project audit records under `.wolio-log/` and
contain plan hash, diff hash, base/result snapshots, actor, affected IDs,
evidence references, status, reviewer, and decision time. The record is created
atomically with POST_REVIEW publication. Agents cannot create an accepted or
rejected disposition. The repository maintainer records the decision.

`pending -> accepted` permits merge/release eligibility. `pending -> rejected`
does not roll back source or later work; a new repair or revert plan is required.
A pending or rejected record cannot be used as baseline acceptance. Review status
is audit metadata and is not included in the source snapshot hash; release
eligibility checks it separately.

## Approval record

HUMAN_REQUIRED approval is bound to the exact plan hash, diff hash, base snapshot,
capabilities, actor scope, approver, expiry and revocation state. It permits one
application only and expires within 24 hours. Approval cannot legalize invalid
syntax, failed verification, or a policy violation.
