`.blueprint/05-quality/RELEASE_CRITERIA.md`
# Release criteria

## M03 trial

Fresh users can install from documented pinned Rust sources, init/check/run/test/
inspect/impact/verify the three examples without LLVM. Contract violations fail
with structured diagnostics. Three change cycles and the pilot have raw evidence.
Limitations and unsupported features are documented. No claims of universal proof.

## Full V1

All M01–M07 tasks accepted. Scoped mutations survive recovery/concurrent-edit
fixtures; native execution matches the interpreter; package integrity/offline
reproduction works; formatter/LSP and clean Linux installation pass. Release
artifacts include checksums, exact tools/lock metadata, license and release notes.
The owner selects the distribution license before T074 publication.

## Evaluation claims

Negative or inconclusive benchmark results do not prevent a technical release.
Improvement claims require DELIVERY_PLAN's statistical and economic thresholds.
Do not omit failures or extrapolate to stateful/concurrent applications unsupported
by V1. Future extension gates are in POST_V1_PLAN, not this release checklist.
