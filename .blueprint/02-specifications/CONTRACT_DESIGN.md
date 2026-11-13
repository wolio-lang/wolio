`.blueprint/02-specifications/CONTRACT_DESIGN.md`
# Behavioral contracts V1

Normative with `language/CORE_V1.md`. A contract belongs to an operation;
separate contract files and interface/implementation inheritance are not V1.

## Obligations

| Component | Obligation | Enforcement |
|---|---|---|
| Signature | callers and implementation agree on nominal types | static |
| requires | caller satisfies predicate on entry | runtime; literal false diagnosed statically |
| ensures | implementation satisfies predicate on normal return | runtime |
| errors | all possible Result Err variants are declared | static conservative analysis |
| effects | transitive effects are contained in declaration | static |
| dependencies | all operation calls are listed | static |
| intent | human purpose, not executable truth | review and linked acceptance tests |

Repeated requires/ensures clauses are conjunctions, evaluated in source order.
Requires sees parameters; ensures additionally sees `result` (the entire return
value, including Result). No `result` in requires. Each predicate must be Bool.
Predicates permit literals, immutable parameter/result fields, builtin/nominal
constructors, operators, if, and exhaustive match. User operation calls are
forbidden, even if pure: this keeps checks finite and reproducible. Contract
evaluation arithmetic faults produce `WOLIO-CONTRACT-004`, not a successful check.

If return type is `Result<T,E>`, E must be a declared enum; `errors` lists its
fully qualified variants. The implementation's possible Err variants must be a
subset. A call returning E conservatively contributes all its declared variants
unless exhaustive matching handles them. Unused declared variants are allowed.
For a non-Result return, errors must be empty. Runtime faults are distinct from
business errors and may not be converted to fake success by generated repairs.

Success and failure guarantees use exhaustive match over `result`:

```wol
ensures match (result) {
    Result::Ok(value) => value >= 0,
    Result::Err(reason) => true
};
```

`true` on an error branch supplies no additional error guarantee. Tests must
still cover each declared business-error path that is reachable under requires.
An invalid external input should normally return a typed validation error;
requires expresses a caller obligation, not a replacement for input validation.

## Composition and evaluation

Arguments evaluate before callee requires; requires runs before the body;
ensures runs after body result and before returning to the caller. Checks remain
enabled in release/native builds. Failed requires is a caller violation;
failed ensures is an implementation violation. A failed check aborts the
invocation; effects already performed are not rolled back. No transaction or
exactly-once behavior is implied. Diagnostics identify clause, operation, caller,
and source span without dumping argument values.

Caller A may rely only on B's declared guarantees. A includes B's effects and
propagated errors, not merely effects directly present in A's text. Static proof
of arbitrary predicate implication is not required. Unknown proof obligations
must retain runtime guards and be reported as runtime-checked.

## Compatibility

Compare old and new signatures, predicates, error/effect/dependency sets.
Strengthening requires or weakening ensures is potentially breaking. Weakening
requires or strengthening ensures can be behavioral refinement, but must be
proved within the supported analysis or classified `unknown`. Added errors,
effects, required dependencies, changed nominal types, removed public symbols,
or changed arity are breaking. Removing an error variant from a public enum
can also break exhaustive client source and is a source-breaking change.

V1 proves identical normalized predicates only; other predicate edits are
`unknown`, never automatically compatible. All public contract changes produce
an old/new semantic diff and dependent/test impact. Public and unknown predicate
changes require owner approval bound to that exact diff. Only private changes
with identical normalized predicates/signatures and unchanged error/effect/
dependency sets qualify for POST_REVIEW under LOCAL_PROTOCOL_V1. Approval permits the specification
change; it cannot make an implementation violation legal.

## Frame conditions and invariants

V1 immutable values have an implicit empty write frame: operations cannot mutate
caller values. Record validity predicates, resource frames, old-state snapshots,
error-state guarantees, lifecycle transitions, and idempotency are reserved for
the stateful extension. They are not accepted syntax in V1. This is a deliberate
boundary, not an unspecified promise of rollback or formal correctness.

## Required conformance cases

Positive: conjunction, nominal inputs, success/error postconditions, pure nested
calls, explicit effects. Negative: wrong predicate type, result in requires,
operation call in predicate, false requires, false ensures, predicate overflow,
undeclared Err variant, hidden transitive effect, missing dependency. Compatibility
fixtures cover all directions above and unknown predicate implication. Native
and interpreter must report the same obligation and failure category.
