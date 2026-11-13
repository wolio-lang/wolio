`.blueprint/01-architecture/decisions/ADR-005-github-package-source.md`
# ADR-005: GitHub-First Package Source

**Status:** Approved

Wolio uses GitHub repositories as the default package source. Repository paths provide globally unambiguous identity, Git tags provide release versions, and commit plus content-hash verification provides reproducibility. A registry is optional discovery infrastructure, not a distribution prerequisite.
