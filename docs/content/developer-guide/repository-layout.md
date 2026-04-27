---
title: Repository Layout
sidebar_position: 3
---

# Repository Layout

A guided tour of the top-level directories.

```
RDFArchitect/
├── backend/                   # Spring Boot application
├── frontend/                  # SvelteKit application
├── docker/                    # Compose + nginx for the local containerised stack
├── docs/                      # This documentation site (Docusaurus)
├── .github/                   # CI workflows, contribution policy, PR templates
├── CHANGELOG.md
├── README.md
└── LICENSE
```

## `backend/`

```
backend/
├── pom.xml                    # Maven build, plugins, dependency management
├── Dockerfile                 # Production image (Temurin 25 JRE)
├── LICENSES-THIRD-PARTY.md    # Generated; CI verifies it
├── config/                    # Tooling config: Checkstyle, Spotless, SpotBugs
└── src/
    ├── main/
    │   ├── java/org/rdfarchitect/
    │   │   ├── api/
    │   │   │   ├── controller/        # @RestController classes
    │   │   │   └── dto/               # Request/response DTOs + MapStruct mappers
    │   │   ├── services/              # Use case interfaces + service implementations
    │   │   ├── database/              # DatabasePort + Fuseki/file/in-memory adapters
    │   │   ├── graph/                 # Graph wrappers, snapshots, transactions
    │   │   ├── rendering/             # Diagram layout
    │   │   ├── shacl/                 # SHACL generator
    │   │   └── migration/             # Migration template composer
    │   └── resources/
    │       ├── application.yml + profile-specific yml files
    │       ├── log4j2.yaml
    │       └── sparql-templates/      # SPARQL fragments used by services
    └── test/java/                     # Controller, service, mapper, and graph tests
```

The backend uses a hexagonal (ports-and-adapters) layout: each feature has a `*UseCase` interface in `services/<feature>/`, an implementation next to it, a `*RESTController` in `api/controller/...`, and DTOs / MapStruct mappers in `api/dto/...`. The `database` package holds the *ports* (`DatabasePort`, `GraphPort`, …) and the *adapters* that bind them to Apache Jena Fuseki, file storage, or an in-memory implementation used in tests.

## `frontend/`

```
frontend/
├── package.json
├── svelte.config.js
├── vite.config.js
├── eslint.config.js
├── eslint-rules/              # Three custom ESLint rules (e.g. license header)
├── nginx.conf                 # nginx config for the production image
├── Dockerfile
├── docker-entrypoint.sh       # Injects PUBLIC_* env vars into static bundle at start
├── docs/script-structure.md   # Convention for the order of <script> blocks in .svelte
├── src/
│   ├── routes/                # SvelteKit pages and route-local components
│   │   ├── +page.svelte               # Welcome page
│   │   ├── mainpage/                  # The editor
│   │   ├── changelog/                 # Per-graph history view
│   │   ├── compare/                   # Diff view
│   │   ├── migrate/                   # 5-step migration wizard
│   │   ├── shacl/                     # SHACL inspection dialogs
│   │   ├── prefixes/
│   │   └── layout/                    # Shared layout primitives
│   ├── lib/                   # Reusable code outside routes
│   │   ├── api/                       # BackendConnection, fetch wrappers
│   │   ├── components/                # Shared UI components
│   │   ├── dialog/                    # bits-ui modal primitives
│   │   ├── eventhandling/             # Global event bus
│   │   ├── models/                    # Domain models (reactive classes)
│   │   ├── rendering/                 # SvelteFlow + custom layouting
│   │   ├── rdf-syntax-grammar/        # Lightweight Turtle/Manchester parsers
│   │   ├── ttl/                       # Turtle helpers
│   │   ├── scripts/                   # Build-time / static helpers
│   │   ├── statePrimitives.svelte.js  # StateValuePair, etc.
│   │   ├── sharedState.svelte.js      # Cross-route reactive state
│   │   ├── styles/
│   │   └── utils/
│   └── app.css                # Tailwind 4 + theme tokens
├── static/                    # Served as-is
└── tests/                     # Vitest tests
```

The frontend is Svelte 5 with runes-based state. Reactive wrappers (`reactive-class`, `reactive-namespace`, …) live in `src/lib/models`. All backend traffic goes through `BackendConnection` — components never construct fetch URLs themselves. Validity rules sit alongside their components.

## `docker/`

```
docker/
├── docker-compose.yaml        # Local 3-service stack (backend, frontend, gateway)
└── nginx.local.conf           # Path rewrites for /api → backend, / → frontend
```

This is the local development stand-in for "production behind a reverse proxy". Production deploys typically replace the gateway with a real ingress (Traefik, Nginx, Apache, …) — see [Access control](/admin-guide/access-control).

## `docs/`

This documentation site, built with Docusaurus. See `docs/README.md` for the dev-loop.

```
docs/
├── docusaurus.config.js
├── sidebars.js
├── package.json
├── content/                   # All Markdown sources
├── static/                    # Images and CNAME for the custom domain
└── src/css/custom.css
```

## `.github/`

```
.github/
├── workflows/
│   ├── backend-ci.yml         # mvn verify on PR
│   ├── frontend-ci.yml        # npm test/lint/build on PR
│   ├── pr-title.yml           # Conventional Commits enforcement
│   ├── publish-test-images.yml
│   └── deploy-docs.yml        # Builds and deploys this site to GitHub Pages
├── ISSUE_TEMPLATE/
├── pull_request_template.md
├── CONTRIBUTING.md            # Pull-request and review policy
├── CODE_OF_CONDUCT.md
├── SECURITY.md
├── SUPPORT.md
├── dco.yml                    # DCO checks
└── renovate.json              # Renovate Bot config
```

The CI workflows verify both apps independently and the docs site as a separate job. Docker images are published via a tag-driven workflow.

## What lives where: a quick map

| Concept | Location |
| ------- | -------- |
| REST endpoints | `backend/src/main/java/org/rdfarchitect/api/controller/` |
| Use cases / services | `backend/src/main/java/org/rdfarchitect/services/` |
| Database ports & adapters | `backend/src/main/java/org/rdfarchitect/database/` |
| SHACL generator | `backend/src/main/java/org/rdfarchitect/shacl/` |
| Migration script composer | `backend/src/main/java/org/rdfarchitect/migration/` |
| SPARQL fragments | `backend/src/main/resources/sparql-templates/` |
| Editor entry point | `frontend/src/routes/mainpage/+page.svelte` |
| Backend-call wrapper | `frontend/src/lib/api/backend.js` |
| Reactive domain models | `frontend/src/lib/models/` |
| Diagram layouting | `frontend/src/lib/rendering/` |
| Shared dialogs | `frontend/src/lib/dialog/` |
| CI pipelines | `.github/workflows/` |
