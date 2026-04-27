---
title: API Stability and Versioning
sidebar_position: 14
---

# API Stability and Versioning

## Semantic versioning

RDFArchitect follows [SemVer 2.0](https://semver.org/) starting with 1.0.0:

- **Major** — breaking change to the REST API surface, the on-disk RDF conventions, or the configuration schema.
- **Minor** — backwards-compatible feature.
- **Patch** — backwards-compatible fix.

The version is sourced from the latest git tag (`vX.Y.Z`) at build time.

## What "API" means

Three surfaces are stable:

1. **The REST API**, as documented by Swagger UI at `/swagger-ui.html`. Adding endpoints is a minor change. Removing or breaking existing endpoints is major.
2. **The RDF conventions**, as documented in [Data Model](./data-model). RDFArchitect promises that any file produced by `vX.*.*` is consumable by `vX.*.*` again. Adding new annotations is minor. Changing the meaning of existing predicates is major.
3. **The configuration schema** in `application*.yml`. Renaming or removing properties is major. Adding optional properties is minor.

The frontend ↔ backend wire format is **not** a separate stability surface — both ship together and only need to agree across the same version.

## What is *not* stable

- Internal Java types and packages — refactoring is fine across any version.
- Internal SPARQL templates — they're implementation details.
- The exact CSS class names or the diagram visual style — those follow the design language, not the version contract.
- The `arch:` namespace vocabulary, *except* for layout, snapshot, and changelog predicates that round-trip through exports.

## Marking a breaking change

Breaking changes require:

- A `feat!:` or `fix!:` PR title (with the `!` to signal breakage).
- A `BREAKING CHANGE:` paragraph in the commit body / PR description explaining what changed and how to migrate.
- A corresponding entry in `CHANGELOG.md` under a `### Breaking Changes` heading.
- A bump of the major version on the next release.

Reviewers explicitly verify the changelog entry exists.

## Pre-1.0 history

Versions before 1.0.0 (as listed in [CHANGELOG.md](https://github.com/SOPTIM/RDFArchitect/blob/main/CHANGELOG.md)) made occasional breaking changes between minor releases. From 1.0 onwards, the rules above apply strictly.

## Deprecations

Mark deprecated endpoints with the OpenAPI `deprecated: true` flag and add a `@Deprecated(forRemoval = true)` annotation in the Java code. Keep deprecated endpoints around for at least one minor release before removing them.

## Frontend ↔ backend skew

In Docker deployments the frontend and backend ship together. In split deployments (e.g. CDN-served frontend, separately-versioned backend), the runtime config injection in `frontend/docker-entrypoint.sh` is the integration point. Pin both to the same version unless you have a specific reason not to.
