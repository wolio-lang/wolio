`.blueprint/02-specifications/SEMANTIC_MODEL.md`
# Semantic model V1

The shared model contains packages, modules, nominal/record/enum types, operations,
implementations, clauses, effects, tests, and source locations. Edges are imports,
calls, depends_on, returns, uses_type, has_contract, has_effect, and tests.
Impact is reverse reachability over typed edges; explain the edge path rather
than claiming that reachability proves behavioral breakage.

## Stable identity

Declaration IDs are `sym_` plus lowercase SHA-256 hex, computed using the
length-prefixed encoding in INTERFACE_CONTRACTS. Fields, in order, are domain
`wolio.symbol.v1`, package identity, module identifier, declaration kind, and
symbol path. Module and symbol names are exact case-sensitive CORE_V1 identifiers;
no filesystem path, signature, source bytes or package version enters this hash.
Kinds are `nominal`, `record`, `enum`, `operation`, `implementation`, and `test`.
Symbol path is the declared name; an implementation uses its operation name and
its distinct kind. Fields/variants use their parent node plus field/variant name
when addressed; local bindings are snapshot-local, not persistent declarations.

Package identity is `git:` plus the canonical repository identity when
`module.path` exists, otherwise `local:` plus the manifest package name. Local
identities are project-scoped, not globally unique; publishing or changing the
package identity changes all declaration IDs and requires an explicit impact
diff. M07 dependencies require repository identities. A standalone frontend
check has no package identity and reports null diagnostic targets.

No `wolio.ids.json`, UUID allocation, ephemeral registration or register-ids
command is required. Formatting, checkout relocation, implementation edits and
file moves preserving module declarations preserve IDs. Rename or move between
modules changes IDs; structured operations record explicit old/new links in
change provenance and update resolved references atomically. Raw edits never
infer lineage from similarity. Duplicate semantic keys fail resolution; distinct
keys yielding the same hash fail checking rather than being merged.

Deletion removes the node. Recreation at the same semantic key deliberately
reuses its ID: this is definition identity, not historical entity identity.
Snapshots and deletion/creation events distinguish incarnations; evidence and
approvals must never be reused based on an ID alone. No identity tombstones or
mutable alias registry are required. Provenance links do not make IDs equivalent
for nominal type checking or authorization.
Contract clauses use operation ID plus normalized clause hash and occurrence
index. Changes produce new clause identities, retaining old/new links in diffs.

Types are interned semantic nodes, not text aliases. Builtin type applications
include their arguments in identity; nominal identity uses declaration ID.
There is no ownership/concurrency model in immutable, sequential V1.

## Determinism and provenance

Queries sort IDs and edges, and carry a source snapshot hash. Snapshots hash
sorted relative source paths and bytes, manifest, lockfile, and
policy; generated cache files are excluded. Hash algorithm is SHA-256.
Change provenance stores intent, actor, base/result hashes, approved diff,
affected IDs, and evidence references. Value provenance is an extension,
distinct from change provenance. Human intent is not treated as a proven fact.
