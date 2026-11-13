`.blueprint/02-specifications/package-manager/PACKAGE_MANAGER_SPEC.md`
# Wolio Package Manager Specification

**Status:** Approved  
**Source model:** GitHub-first, Git-native

This document incorporates the approved package-management decision. Wolio does not require a central package registry: a GitHub repository, immutable release tag, and `wolio.toml` manifest are sufficient to publish and consume a package.

## 1. Package identity and source

The canonical identity of a package is its repository URL path, normally `github.com/<owner>/<repository>`. The repository is the primary source of truth; no registration step is required. GitLab, Gitea, self-hosted Git servers, and GitHub Enterprise may be supported through the same VCS interface.

The package manifest is `wolio.toml`:

```toml
[package]
name = "http"
version = "1.2.0"
license = "MIT"

[module]
path = "github.com/wolio-community/http"

[dependencies]
"github.com/wolio-community/crypto" = "2.1.0"
```

`module.path` must match the repository identity. Releases use Semantic Versioning and immutable Git tags are strongly preferred.

## 2. Resolution and integrity

The resolver fetches repository metadata, selects a compatible SemVer tag, resolves transitive dependencies, and produces a deterministic graph. Every resolved package must be recorded in `wolio.lock` with its version, repository, tag, commit, source hash, dependency hash, contract hash, and compiler compatibility.

Install succeeds only when the expected commit and source hash match the resolved content. A moved tag or changed source is rejected with `WOLIO-PACKAGE-INTEGRITY-001`; the manager must never silently accept it.

## 3. CLI

```text
wolio add github.com/wolio-community/http@v1.2.0
wolio remove github.com/wolio-community/http
wolio install
wolio update [github.com/wolio-community/http]
wolio list
wolio info github.com/wolio-community/http
wolio graph
```

## 4. Authentication and cache

Private repositories use the user's configured Git credentials, SSH agent, credential helper, or GitHub Enterprise configuration. Tokens must never be stored in `wolio.toml`, `wolio.lock`, or source code.

The user cache is content-addressed under `~/.wolio/cache/`. The cache accelerates builds but is not authoritative; `wolio.lock` remains the source of the selected dependency set.

## 5. Public API boundary

Packages must explicitly define their public API. Internal implementation details are unavailable to consumers unless exported by the package contract. Dependency changes must participate in semantic impact analysis.

## 6. V1 deterministic resolution (M07)

Constraints support exact SemVer or caret ranges only; prereleases require an
explicit exact version. One version per repository identity; conflicting ranges
fail with the dependency chains. Resolve sorted repository identities, trying
highest matching versions first with deterministic backtracking; dependency
cycles fail. Lockfile schema version 1 is canonical sorted TOML. Ordinary install
uses the existing lock, never upgrades it; update explicitly re-resolves.

Repository identity is lowercase host plus case-preserving owner/repository path,
without `.git` suffix or credentials. Only HTTPS and SSH Git transports are allowed.
Source hash is SHA-256 over sorted relative tracked-file paths, executable bit,
length and bytes at the pinned commit, excluding `.git`; reject symlinks escaping
the package and submodules in V1. Contract hash uses normalized public signatures
and predicates. Dependency hash covers sorted child identities and lock entries.
Build scripts are unsupported. Cache hits must pass integrity verification.
Offline missing objects fail explicitly. Tags are not inherently immutable: a
locked tag observed at a different commit fails instead of rewriting the lock.
