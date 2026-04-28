# RDFArchitect — Developer Guide

A guide for developers who want to **contribute** to RDFArchitect, **extend it**, or **integrate it** into other systems. If you only want to use the application, the [User Guide](user-guide.md) is the right place.

This guide complements [`.github/CONTRIBUTING.md`](../.github/CONTRIBUTING.md), which is the authoritative source on PR rules, commit format, and review process. Read that first; this document fills in the *how* behind those rules.

---

## Table of Contents

1. [Getting set up](#1-getting-set-up)
2. [Repository layout](#2-repository-layout)
3. [Backend architecture](#3-backend-architecture)
4. [Frontend architecture](#4-frontend-architecture)
5. [Adding a feature end-to-end](#5-adding-a-feature-end-to-end)
6. [Testing](#6-testing)
7. [Code style and quality gates](#7-code-style-and-quality-gates)
8. [API stability and versioning](#8-api-stability-and-versioning)
9. [Working with RDF, SHACL, and SPARQL inside the codebase](#9-working-with-rdf-shacl-and-sparql-inside-the-codebase)
10. [CI, releases, and Docker images](#10-ci-releases-and-docker-images)
11. [Common contribution scenarios](#11-common-contribution-scenarios)
12. [Where to ask questions](#12-where-to-ask-questions)

---

## 1. Getting set up

### Required toolchain

| Tool        | Minimum version | Notes                                                         |
| ----------- | --------------- | ------------------------------------------------------------- |
| Java        | 25              | Temurin is what CI uses; any compatible JDK works locally.    |
| Maven       | 3.9.9           | The project does not use the Maven wrapper.                   |
| Node.js     | 24              |                                                               |
| npm         | 11              |                                                               |
| Docker      | recent          | Optional, only needed for the Compose-based local stack.      |
| Apache Jena Fuseki | 5.x      | Required at runtime — see [Installation](installation.md).    |

A working Fuseki at `http://localhost:3030` with a writable dataset called `default` is the simplest way to develop locally. The [installation guide](installation.md#fuseki-quickstart) has a one-line Docker invocation.

### Clone, build, run

```bash
git clone https://github.com/SOPTIM/RDFArchitect.git
cd RDFArchitect

# Backend (terminal 1)
cd backend
mvn spring-boot:run

# Frontend (terminal 2)
cd frontend
npm install
npm run dev
```

Open `http://localhost:1407`. Swagger UI for the backend is at `http://localhost:8080/swagger-ui.html`.

### IDE setup

- **IntelliJ IDEA** (Community works) for the backend. Import as a Maven project. Enable the Lombok plugin and "Annotation Processing" — Lombok and MapStruct both rely on it.
- **VS Code** with the Svelte and ESLint extensions for the frontend. The repository ships with the lint config; no per-machine setup required.
- **Pre-commit hook (optional)**: a quick `mvn -B spotless:apply && cd ../frontend && npm run format` before committing avoids most CI lint failures.

### Hot reload

- **Backend**: Spring Boot DevTools is *not* on the classpath. Restart Maven for changes. For tighter loops, run individual tests with `mvn -B test -Dtest=ClassName`.
- **Frontend**: Vite hot-reloads on save. Type changes in `.ts` files require a tab refresh occasionally; component changes do not.

---

## 2. Repository layout

```
.
├── backend/                  # Spring Boot REST service (Java 25, Maven)
│   ├── src/main/java/org/rdfarchitect/
│   │   ├── api/              # REST layer: controllers + DTOs
│   │   ├── services/         # Use cases and service implementations
│   │   ├── database/         # Database port + Fuseki/in-memory adapters
│   │   ├── rdf/              # RDF/Jena helpers (graph wrappers, formatting, merging)
│   │   ├── shacl/            # SHACL generation engine
│   │   ├── dl/               # Diagram-layout persistence (positions, sizes)
│   │   ├── models/           # Domain models (CIM, changelog, search)
│   │   ├── exception/        # Application exceptions and Spring handlers
│   │   ├── config/           # Spring configuration beans
│   │   ├── filters/          # Servlet filters (CORS, session)
│   │   ├── listeners/        # Spring event listeners
│   │   └── Launcher.java     # Spring Boot entry point
│   └── src/main/resources/
│       ├── application*.yml  # Configuration
│       └── sparql-templates/ # Parameterised SPARQL used by services
│
├── frontend/                 # SvelteKit single-page app (Svelte 5 runes)
│   └── src/
│       ├── routes/           # SvelteKit pages (homepage, /mainpage, /changelog, /compare, /migrate, /shacl)
│       └── lib/
│           ├── api/          # BackendConnection + small fetch helpers
│           ├── components/   # Reusable UI primitives + bits-ui wrappers
│           ├── models/       # DTO types and reactive wrappers ($state-backed)
│           ├── rendering/    # SvelteFlow + Mermaid diagram renderers
│           ├── dialog/       # Generic dialog scaffolding
│           ├── ttl/          # CodeMirror-based TTL editor with validation
│           ├── rdf-syntax-grammar/  # IRI/prefix/NCName validation rules
│           └── sharedState.svelte.js # Cross-component reactive state
│
├── docker/                   # Compose-based local stack (gateway + frontend + backend)
├── docs/                     # User-facing documentation (this folder)
├── .github/                  # CI workflows, issue/PR templates, governance
└── CHANGELOG.md              # Manually maintained, Keep-a-Changelog style
```

The split between backend and frontend is **strict**: nothing under `backend/` imports from `frontend/`, and vice versa. The contract between them is the REST API, documented in Swagger UI at runtime.

---

## 3. Backend architecture

The backend follows a deliberate **hexagonal / ports-and-adapters** layout. The dependency direction is always *outer → inner*:

```
   ┌──────────────────────────────────────────────────────────┐
   │ api/controller/        ← HTTP boundary (Spring MVC)      │
   │ api/dto/               ← request/response shapes         │
   │     │                                                    │
   │     ▼                                                    │
   │ services/<feature>/    ← use cases (interfaces) +        │
   │                          their implementations           │
   │     │                                                    │
   │     ▼                                                    │
   │ database/DatabasePort  ← port interface                  │
   │ database/inmemory/     ┐                                 │
   │ database/implementations/http/  ← Fuseki adapter         │
   │ database/implementations/file/  ← file adapter           │
   └──────────────────────────────────────────────────────────┘
```

### Use case interfaces

Every action exposed by the application is a one-method `*UseCase` interface, e.g.:

```java
public interface ListDatasetsUseCase {
    List<String> listDatasets();
}
```

A controller depends only on the use case interface; a service implements one or many use cases. This is a deliberate design choice — it keeps controllers small, makes individual operations easy to test, and lets services compose multiple ports without becoming god classes.

When you find yourself writing a private helper method on a controller that does any actual work, that's a signal it should be a new use case instead.

### Services

Services live under `services/<feature>/` and typically implement multiple use cases when they share state, repositories, or transaction boundaries. The `services/select/QueryDatasetService` is a good representative example — it implements `GetDatasetSchemaUseCase`, `ListGraphsUseCase`, `ListPrefixesUseCase`, and `ListDatasetsUseCase`, all of which need the same `DatabasePort`.

### The database port

`DatabasePort` is the only direct contact with persistent storage. There are two adapters:

- **`database/implementations/http`** — talks to Fuseki over the SPARQL 1.1 protocol + Graph Store Protocol. The default in production.
- **`database/implementations/file`** — reads/writes TriG or N-Quads files on disk. Development-only.

Plus an **in-memory** path (`database/inmemory`) used heavily in tests and as a per-session working buffer for unsaved edits.

### REST controllers

Controllers are thin and follow a strict skeleton:

```java
@RestController
@RequestMapping("api/datasets")
@RequiredArgsConstructor
public class DatasetRESTController {

    private static final Logger logger = LoggerFactory.getLogger(...);

    private final ListDatasetsUseCase listDatasetsUseCase;
    private final DeleteDatasetUseCase deleteDatasetUseCase;

    @Operation(summary = "...", description = "...", tags = {...})
    @GetMapping
    public List<String> listDatasets(...) {
        logger.info("Received GET request: ...");
        var result = listDatasetsUseCase.listDatasets();
        logger.info("Sending response to GET ...");
        return result;
    }
}
```

Conventions worth knowing:

- **One controller per resource path tier.** `api/datasets`, `api/datasets/{name}/graphs`, `api/datasets/{name}/graphs/{uri}/classes`, etc. Controllers do not span tiers.
- **Always `@Operation` annotated** — Swagger UI is part of the public deliverable.
- **Always log on receive and on respond**, with the originating URL, dataset, and graph names where applicable. Audit log style.
- **Use cases are constructor-injected via `@RequiredArgsConstructor` (Lombok).** No field injection.
- **Read the `Origin` header into a parameter called `originURL`.** It is logged but never used for authorisation — auth lives outside the application.

### DTOs and mapping

DTOs live under `api/dto/` and are organised by feature (`attributes/`, `associations/`, `enumentries/`, `migration/`, `ontology/`, `packages/`, `rendering/`). They are flat data carriers — Lombok `@Value` or `@Data` — with no behaviour.

**MapStruct** generates DTO ↔ domain mappers. When you add a new DTO, also add the corresponding `*Mapper` interface and let MapStruct generate the implementation. The annotation processor runs as part of `mvn compile`.

### Exception handling

Domain exceptions live in `exception/<area>/` and are translated to HTTP responses by handlers in `exception/handlers/`. A new exception that needs a non-500 response *must* have a handler — there is no fallback that turns arbitrary exceptions into 4xx.

### SPARQL templates

Parameterised SPARQL queries live in `src/main/resources/sparql-templates/` and are loaded by classpath utility methods. The migration use cases use them heavily — see `sparql-templates/migration/*.sparql` for the templates that the wizard composes into the final UPDATE script. Keep templates here rather than inline string concatenation in Java.

---

## 4. Frontend architecture

### Stack

- **SvelteKit** (Svelte 5 with runes — `$state`, `$derived`, `$effect`, `$props`, `$bindable`).
- **Vite** for build and dev server.
- **Tailwind CSS** for styling, with project-specific design tokens (CSS variables) in `src/lib/styles/`.
- **bits-ui** for headless dialog, menubar, and dropdown primitives, wrapped in project-local components under `src/lib/components/bitsui/`.
- **@xyflow/svelte** (SvelteFlow) for diagram rendering, with **elkjs** for auto-layout.
- **Mermaid** as the alternative renderer.
- **CodeMirror 6** with `codemirror-lang-turtle` for the TTL editors.
- **Asciidoctor.js** for rendering class comments.
- **Vitest** for unit tests, **jsdom** for the DOM environment.

### Routes

SvelteKit's filesystem routing is used straightforwardly:

```
/                  →  Homepage (routes/+page.svelte)
/mainpage          →  Main editor (left tree + diagram + right class editor)
/changelog         →  Edit history view
/compare           →  Compare results view
/migrate           →  5-step migration wizard
/shacl/...         →  SHACL views
```

The editor listens to URL parameters `?dataset=...&graph=...&package=...` to support deep links.

### Reactive models

A central pattern: every editable domain object has a **reactive wrapper** in `lib/models/reactive/models/` (e.g. `reactive-class.svelte.js`, `reactive-namespace.svelte.js`, `reactive-ontology.svelte.js`). These wrappers:

- Hold the original DTO and a working copy.
- Track `isModified`, `isValid`, and per-field violations as `$derived` runes.
- Expose `save()`, `reset()`, and field accessors.
- Drive the inline validation visible in dialogs.

**DTO ↔ reactive object mapping** lives in `lib/models/reactive/mapper/`. Whenever you add a new field to the backend that the frontend needs to edit, the chain to update is:

1. Backend DTO.
2. Frontend type in `lib/models/dto/`.
3. Reactive wrapper in `lib/models/reactive/models/`.
4. DTO ↔ reactive mapper.
5. UI component that exposes the field.

### Validity rules

`lib/models/reactive/validity-rules/validityFunctions.js` is the single home for cross-cutting validation (label uniqueness, prefix uniqueness, valid IRIs, valid NCNames, etc.). Reuse what's there before adding a new function.

### Backend communication

`BackendConnection` in `lib/api/backend.js` is a hand-written class with one method per backend endpoint. Every method:

- Builds the URL from `PUBLIC_BACKEND_URL`.
- Uses `credentials: "include"` so the session cookie travels.
- Returns the raw `Response` — callers decide whether to `.json()`, `.text()`, or `.blob()`.

When you add a new backend endpoint, add the matching method here. Do not call `fetch()` directly from a component — the indirection makes mocking in tests possible and keeps URLs in one place.

### Shared state

`lib/sharedState.svelte.js` exports cross-component reactive state, primarily:

- `editorState.selectedDataset` / `selectedGraph` / `selectedPackageUUID` / `selectedClassUUID`
- `forceReloadTrigger` — a "kick everything to refresh" signal used after destructive actions
- `compareState`, `migrationState` — wizard state passed across pages

Use these instead of inventing per-component prop drilling for global selections.

### Svelte script ordering

The repository enforces a specific script-block ordering (imports → props → constants → state → derived → effects → lifecycle → functions). The full rationale and ESLint rule are in [`frontend/docs/script-structure.md`](../frontend/docs/script-structure.md). The custom rule lives at `frontend/eslint-rules/rules/svelte-script-order/`. When in doubt, run `npm run format` and follow whatever the auto-fixer produces.

### Custom ESLint rules

Three custom rules ship with the project:

- `copyright-header` — enforces the Apache 2.0 header on every source file.
- `svelte-file-structure` — order of `<script>`, markup, `<style>`.
- `svelte-script-order` — order *inside* `<script>` blocks.

If you write a new file, the auto-fixer (`npm run format`) inserts the header for you.

---

## 5. Adding a feature end-to-end

A short worked example: suppose you want to add an endpoint that lists all *abstract* classes in a graph and surface them in the navigation tree with an italic style.

### Step 1 — Backend use case

```java
// services/select/ListAbstractClassesUseCase.java
public interface ListAbstractClassesUseCase {
    List<ClassDTO> listAbstractClasses(GraphIdentifier graphIdentifier);
}
```

### Step 2 — Service implementation

Either extend an existing service (if one already holds the same `DatabasePort` and is logically related — e.g. `QueryClassService`) or create a new `services/select/AbstractClassQueryService` that implements the use case and wires the `DatabasePort` via `@RequiredArgsConstructor`.

### Step 3 — Controller endpoint

Add a new `@GetMapping` to `AllClassesRESTController` (or a new controller if a new path tier is involved). Keep the controller thin — call the use case, log on receive and respond.

```java
@GetMapping("/abstract")
@Operation(summary = "list abstract classes", tags = {"class"})
public List<ClassDTO> listAbstract(@PathVariable String datasetName,
                                    @PathVariable String graphURI,
                                    @RequestHeader(...) String originURL) {
    logger.info("Received GET request: \".../abstract\" from \"{}\".", originURL);
    var result = listAbstractClassesUseCase.listAbstractClasses(...);
    logger.info("Sending response to GET request: \".../abstract\" to \"{}\".", originURL);
    return result;
}
```

### Step 4 — Tests

- Unit-test the service against the in-memory database (`InMemoryDatabaseImpl`).
- Integration-test the endpoint with `@SpringBootTest` + `MockMvc` if the routing or DTO shape is non-trivial.

### Step 5 — Frontend BackendConnection method

```javascript
async listAbstractClasses(datasetName, graphURI) {
    const url = `${PUBLIC_BACKEND_URL}/datasets/${encodeURIComponent(datasetName)}/graphs/${encodeURIComponent(graphURI)}/classes/abstract`;
    return fetch(url, {
        method: "GET",
        credentials: "include",
    });
}
```

### Step 6 — Reactive consumer

Either update the existing class-list reactive store or read on demand from a component. If the navigation tree needs to mark the abstract ones, extend `build-nav-object.js` to include the abstract flag on each class node, and update `ClassEntry.svelte` to render italic when the flag is set.

### Step 7 — Tests

Vitest unit tests for any new logic in `lib/`. Manual smoke test through the UI against a live Fuseki instance.

### Step 8 — PR

Follow the squash commit format and the pre-merge checklist in [`CONTRIBUTING.md`](../.github/CONTRIBUTING.md). Update `CHANGELOG.md` under `## [Unreleased]` if the change is user-facing.

---

## 6. Testing

### Backend

- **Unit tests** — `mvn -B test`. Surefire runs everything matching `*Test.java`.
- **Integration tests** — `mvn -B verify`. Failsafe runs everything matching `*IT.java`. Integration tests typically spin up Spring with `@SpringBootTest`.
- **Coverage** — JaCoCo runs as part of `verify`. The report is at `backend/target/site/jacoco/index.html`. There is no enforced threshold; aim for new code to be at least as well-covered as the area it sits in.
- **Test resources** — RDF fixtures live under `src/test/resources/`. Tests that need a triple store use `InMemoryDatabaseImpl`, which is faster and isolated.

The largest test suites today are around CIM-graph-to-DTO conversion (`cim/data/...`) and SHACL generation (`shacl/SHACLFromCIMGeneratorTest`); these are the canonical references for "what does a good RDFArchitect test look like".

### Frontend

- **Unit tests** — `npm run test` (Vitest, jsdom). Component-level and pure-function tests under `tests/`.
- **Linting** — `npm run lint` runs Prettier check, ESLint, and a third-party-license consistency check.
- **Build** — `npm run build` is run in CI; if it breaks, the lint did not catch the issue.

There is no end-to-end browser test suite at the time of writing. The `.github/pull_request_template.md` includes a substantial **manual test checklist** that contributors are expected to walk through before requesting review — treat it as the de-facto integration test.

### Running everything locally before a PR

```bash
# Backend
cd backend
mvn -B spotless:apply             # auto-format
mvn -B -Plint -DskipTests verify  # checkstyle, spotbugs, license headers
mvn -B test
mvn -B verify

# Frontend
cd ../frontend
npm run format                    # auto-format (prettier + eslint --fix)
npm run lint
npm run test
npm run build
```

CI runs all of the above on every PR. Running them locally first is much faster than waiting for the CI feedback loop.

---

## 7. Code style and quality gates

### Backend

| Tool                  | Purpose                                                     | Run                                  |
| --------------------- | ----------------------------------------------------------- | ------------------------------------ |
| **Spotless**          | google-java-format (AOSP), import order, trailing whitespace| `mvn -B spotless:apply`              |
| **Checkstyle**        | Style rules beyond formatting                               | `mvn -B -Plint -DskipTests verify`   |
| **SpotBugs**          | Bug pattern detection                                       | `mvn -B -Plint -DskipTests verify`   |
| **Mycila license-maven-plugin** | Apache 2.0 header on every Java file              | `mvn -B -Plint -DskipTests verify`   |
| **codehaus license-maven-plugin** | Third-party license aggregation                 | `mvn org.codehaus.mojo:license-maven-plugin:add-third-party` |

The `lint` Maven profile runs everything in one go without tests.

### Frontend

| Tool                          | Purpose                                                                  |
| ----------------------------- | ------------------------------------------------------------------------ |
| **Prettier**                  | Formatting (with `prettier-plugin-svelte` and `prettier-plugin-tailwindcss`) |
| **ESLint** (custom config)    | Lint rules, including project-specific custom rules                      |
| **Custom ESLint rules**       | Copyright header, Svelte file structure, Svelte script order             |
| **`licenses-third-party.js`** | Third-party license aggregation                                          |

Both `npm run lint` and `npm run format` run all three together.

### License header

Every source file (Java, JavaScript, TypeScript, Svelte) must carry the Apache 2.0 header. The auto-fixers add it for you. CI fails if it is missing.

### Renovate

The repository uses Renovate for dependency updates. CI is set up so that Renovate's PRs auto-regenerate `LICENSES-THIRD-PARTY.md` if a dependency change requires it. Human contributors do not need to do this manually — but if you change `pom.xml` or `package.json`, you may need to run the appropriate licenses task locally and commit the result.

---

## 8. API stability and versioning

### Versioning scheme

Semantic Versioning, derived from git tags (`vX.Y.Z`):

- **Patch** (`v1.0.x`) — bug fixes, internal changes, no behaviour change.
- **Minor** (`v1.x.0`) — new features, backward-compatible API changes.
- **Major** (`vX.0.0`) — breaking changes.

The version shown in the application is computed at build time from `git describe` — see `.github/scripts/resolve_version.sh`. Local builds without a tag in their history get `0.0.0-SNAPSHOT`.

### What counts as breaking?

- Removing a REST endpoint.
- Removing a field from a request or response DTO.
- Changing the type or semantics of an existing field.
- Changing required headers or authentication behaviour.
- Renaming a configuration property.
- Anything that requires existing custom SHACL or stored data to be migrated.

Mark such changes with `!` in the conventional-commit header (`feat(api)!: ...`) or `BREAKING CHANGE:` in the body. Update `CHANGELOG.md` under a `### Breaking Changes` section.

### What does *not* count as breaking?

- Adding a new endpoint.
- Adding a new optional field to an existing response.
- Adding a new optional request parameter that has a sensible default.
- Internal refactors that do not change the REST surface.

### REST surface and Swagger

Swagger UI is the public-facing documentation of the REST API. Every endpoint must be `@Operation`-annotated; every DTO field that has a non-obvious meaning should carry an `@Schema` description. CI does not currently enforce this, but reviewers do.

---

## 9. Working with RDF, SHACL, and SPARQL inside the codebase

### Apache Jena

The backend uses **Apache Jena 5.x** end-to-end. The relevant entry points are:

- `org.apache.jena.rdf.model.Model` — the high-level RDF API. Used in services for read-side work.
- `org.apache.jena.graph.Graph` — the low-level triple-set API. Used in `database/`, `rdf/`, and the in-memory adapter where performance matters.
- `org.apache.jena.query.QueryFactory` / `UpdateAction` — for SPARQL.
- `org.apache.jena.shacl.*` — for SHACL evaluation when needed.

The `rdf/` package contains the project's own helpers (graph wrappers with version history, RDF formatting that matches ENTSO-E conventions, model merging utilities). Use these instead of Jena's defaults where they exist — they encode CIM-specific output decisions (resource ordering, prefix handling, etc.) that downstream tooling expects.

### SHACL generation

SHACL shapes are generated procedurally from CIM model objects. The entry point is `services/shacl/SHACLGenerateService` (use case: `SHACLGenerateUseCase`), which delegates to the builders under `shacl/property/shapegenerator/` and `shacl/property/shapebuilder/`. Each property type (attribute, association, enum-typed) has its own builder.

When fixing a SHACL bug, the most useful starting point is `SHACLFromCIMGeneratorTest`. It loads a known-good CIM graph and asserts the shape of the generated SHACL — adding a failing case there is the quickest way to reproduce.

### Diagram layout

Layout positions are persisted as RDF using a small custom vocabulary under `dl/rdf/` (Diagram Layout). Layout DTOs live in `api/dto/rendering/` with renderer-specific subdirectories (`svelteflow/`, `mermaid/`). When changing the layout schema, update both the `dl/` model and any code that reads or writes layout data.

### Migration scripts

The migration generator stitches together templates from `src/main/resources/sparql-templates/migration/`. Each template is a parameterised SPARQL UPDATE block — `class-renamed.sparql`, `attribute-renamed.sparql`, etc. The composer is in `services/schemamigration/`. Adding a new migration capability means: (1) adding the template, (2) wiring it into the composer, and (3) extending the wizard's confirmation step DTOs and UI.

---

## 10. CI, releases, and Docker images

### Workflows

Two GitHub Actions workflows run on every push and PR:

- **`.github/workflows/backend-ci.yml`** — checkout, setup Java 25, resolve version, lint (`mvn -B -Plint -DskipTests verify`), unit tests (`mvn -B test`), integration tests (`mvn -B verify`), third-party license consistency check, build & push Docker image (only on `main` and tags).
- **`.github/workflows/frontend-ci.yml`** — checkout, setup Node 24, install, lint, test, build, third-party license check, build & push Docker image (only on `main` and tags).

A third workflow, **`publish-test-images.yml`**, builds preview images for PRs from trusted contributors.

### Releases

Releases are tag-driven:

1. Update `CHANGELOG.md`: move items from `## [Unreleased]` into a new `## [X.Y.Z] - YYYY-MM-DD` section.
2. Push a tag of the form `vX.Y.Z` to `main`.
3. CI builds and publishes Docker images tagged `X.Y.Z`, `X.Y`, and `X` (with `X` suppressed for `0.x.y`) plus `latest` on the default branch.
4. A GitHub Release is created automatically.

### Docker images

Images are published to **GHCR**:

- `ghcr.io/soptim/rdfarchitect-backend`
- `ghcr.io/soptim/rdfarchitect-frontend`

The compose file in `docker/` builds them locally; you can substitute the published images by changing `build:` to `image:`.

---

## 11. Common contribution scenarios

### Fixing a UI bug

Almost always confined to `frontend/src/`. Reproduce against a local backend, write a Vitest unit test if the bug is in pure logic, fix, run `npm run lint && npm run test && npm run build`, walk the relevant section of the manual test checklist, open a PR.

### Adding a new field to a class / attribute / association

Touches every layer: backend DTO → frontend DTO type → reactive wrapper → mapper → UI control → tests on both sides. Use the [end-to-end walkthrough](#5-adding-a-feature-end-to-end) above as a template. Keep all changes in one PR if scoped to a single field.

### Fixing or extending SHACL generation

Backend-only. The starting point is `services/shacl/SHACLGenerateService` and its test, `SHACLFromCIMGeneratorTest`. Add a failing test case first; it makes the fix easier to verify and prevents regressions.

### Adding migration support for a new schema-change pattern

Three steps:

1. Add a SPARQL template under `src/main/resources/sparql-templates/migration/`.
2. Extend `services/schemamigration/` to detect and emit the new pattern.
3. Extend the wizard's relevant step (`frontend/src/routes/migrate/steps/`) to confirm or override the proposed change.

### Improving documentation

Documentation lives in `docs/`. Markdown files there are referenced from the README and from each other. Screenshots go in `docs/assets/screenshots/`. PRs that improve documentation are *especially* welcome — they are the lowest-effort way to start contributing and they help everyone.

### Internationalisation

Not currently supported. Contributions that introduce a translation framework would be a meaningful change to the frontend architecture and are best discussed in an issue first.

---

## 12. Where to ask questions

- **Architecture or design questions**: open a discussion in [GitHub Discussions](https://github.com/SOPTIM/RDFArchitect/discussions). Maintainers prefer this channel for anything that does not yet correspond to a concrete bug or feature.
- **Bugs**: open an issue with the bug report template. Include the version (visible on the homepage), browser, and a reproducer.
- **Features**: open an issue with the feature request template, or — better — open a draft PR to start the conversation around concrete code.
- **Security issues**: do not use public channels. See [`SECURITY.md`](../.github/SECURITY.md).

---

## Appendix: useful commands

```bash
# Backend — common single commands
mvn -B spotless:apply                       # auto-format Java
mvn -B -Plint -DskipTests verify            # all linters (checkstyle, spotbugs, license)
mvn -B test                                 # unit tests
mvn -B verify                               # full verify (unit + integration + jacoco)
mvn -B test -Dtest=ClassName                # single test class
mvn -B test -Dtest=ClassName#methodName     # single test method

# Frontend — common single commands
npm run dev                                 # dev server on :1407
npm run build                               # production build
npm run test                                # vitest
npm run test -- --watch                     # vitest in watch mode
npm run lint                                # prettier + eslint + license check
npm run format                              # prettier --write + eslint --fix
npm run clean-install                       # nuke node_modules and reinstall

# Docker — full stack
cd docker
docker compose up --build                   # build + run
docker compose down                         # stop + remove
docker compose up backend                   # only the backend (frontend at :1407 expected)
```

Welcome to RDFArchitect — looking forward to your contribution.
