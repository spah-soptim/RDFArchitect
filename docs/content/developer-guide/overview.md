---
title: Developer Guide Overview
sidebar_position: 1
---

# Developer Guide

This guide is for engineers who plan to **read, modify, or extend the RDFArchitect codebase**. If you only want to use the application, see the [User Guide](/user-guide/overview).

## What's in this guide

1. **[Setup](./setup)** — toolchain, clone, build, run.
2. **[Repository layout](./repository-layout)** — annotated tree of `backend/`, `frontend/`, `docker/`, `docs/`, `.github/`.
3. **[Run and debug](./run-and-debug)** — running the two services, hot-reload, IDE setup, common pitfalls.
4. **[Backend architecture](./backend-architecture)** — Spring Boot, hexagonal pattern, ports and adapters, Jena/Fuseki integration.
5. **[Frontend architecture](./frontend-architecture)** — Svelte 5 runes, route map, reactive wrappers, the `BackendConnection` indirection.
6. **[Data model](./data-model)** — RDFS/OWL/SHACL conventions used in the graph.
7. **[Adding a feature end-to-end](./adding-a-feature)** — walked through all eight layers with a concrete example.
8. **[RDF, SHACL, SPARQL](./rdf-shacl-sparql)** — Jena entry points, SHACL generator structure, migration template composer.
9. **[Testing](./testing)** — backend unit/integration patterns, frontend Vitest, CI gates.
10. **[Style and quality gates](./style-and-quality-gates)** — Spotless, Checkstyle, SpotBugs, Prettier, ESLint, custom rules.
11. **[Dependencies](./dependencies)** — Renovate, license aggregation.
12. **[CI and releases](./ci-and-releases)** — workflows, GHCR images, tag-driven releases.
13. **[API stability](./api-stability)** — semver, breaking-change policy, Swagger as the public surface.
14. **[Contribution scenarios](./contribution-scenarios)** — recipes for fix-a-UI-bug, add-a-field, fix-SHACL, add-migration-pattern, docs-only.

## Reading map by intent

| You want to… | Start at |
| ------------ | -------- |
| Make your first contribution | [Setup](./setup) → [Adding a feature](./adding-a-feature) |
| Understand the backend | [Backend architecture](./backend-architecture) |
| Understand the frontend | [Frontend architecture](./frontend-architecture) |
| Add a SHACL feature | [RDF, SHACL, SPARQL](./rdf-shacl-sparql) |
| Add a migration template | [Migration wizard](/user-guide/migration-wizard) → [RDF, SHACL, SPARQL](./rdf-shacl-sparql) |
| Investigate why CI is red | [CI and releases](./ci-and-releases) |
| Update a dependency | [Dependencies](./dependencies) |

## How to ask questions

Open a GitHub Discussion or Issue: [github.com/SOPTIM/RDFArchitect](https://github.com/SOPTIM/RDFArchitect). Pull requests are welcome — see [`.github/CONTRIBUTING.md`](https://github.com/SOPTIM/RDFArchitect/blob/main/.github/CONTRIBUTING.md) for the policy.
