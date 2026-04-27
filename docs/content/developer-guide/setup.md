---
title: Setup
sidebar_position: 2
---

# Setup

Bring up a working RDFArchitect development environment in roughly fifteen minutes.

## Toolchain

| Tool | Version | Notes |
| ---- | ------- | ----- |
| Java (JDK) | **25** or higher | Eclipse Temurin works; pinned in `backend/pom.xml`. |
| Maven | **3.9.9** or higher | Build tool for the backend. |
| Node.js | **24** or higher | Frontend toolchain. |
| npm | **11** or higher | Bundled with Node 24+. |
| Git | recent | |
| Docker + Docker Compose | optional | Only needed for the containerised local stack. |
| Apache Jena Fuseki | optional | Only needed for non-default storage; in-memory mode works out of the box for development. |

A working IDE makes life nicer:

- **IntelliJ IDEA** (Ultimate or Community) for the backend. The repo includes `.idea/` configuration for codestyle and inspections.
- **VS Code** with the Svelte extension for the frontend.

## Clone

```bash
git clone https://github.com/SOPTIM/RDFArchitect.git
cd RDFArchitect
```

## Backend

### Build

```bash
cd backend
mvn -B verify
```

`mvn verify` is the gate that CI runs. Locally it formats with Spotless, runs Checkstyle and SpotBugs, runs the test suite, and produces the Spring Boot fat jar.

### Run

```bash
mvn spring-boot:run
```

The backend listens on `:8080`. Spring Boot DevTools is configured for hot reload — recompiling the project (e.g. `mvn compile` from another shell, or saving in IntelliJ with auto-make) will trigger a restart.

### Default storage

By default the backend talks to a Fuseki endpoint at `http://localhost:3030`. For the lowest-friction first run, start a Fuseki container in another terminal:

```bash
docker run --rm -p 3030:3030 \
  -e ADMIN_PASSWORD=admin \
  stain/jena-fuseki:5
```

Or switch the backend to file storage by editing `backend/src/main/resources/application-database.yml` (set `databaseType: file` and pick a path).

### Configuration

Spring config files live under `backend/src/main/resources`:

- `application.yml` — main file; imports the others.
- `application-database.yml` — datastore.
- `application-frontend.yml` — `frontend.url`, CORS access route.
- `application-graph.yml` — graph constants, default datasets.
- `application-rendering.yml` — diagram layout defaults.

Override anything via environment variables (`SPRING_PROFILES_ACTIVE`, `DATABASE_HTTP_ENDPOINT`, `FRONTEND_URL`, …) or with a `--spring.config.additional-location=` flag.

## Frontend

### Install dependencies

```bash
cd frontend
npm run clean-install
```

`clean-install` removes `node_modules` and `package-lock.json`'s install state and reinstalls — use it whenever you switch branches with diverging dependencies.

### Dev server

```bash
npm run dev
```

The Vite dev server runs on `http://localhost:1407`. It expects the backend at `http://localhost:8080` (configurable via `PUBLIC_BACKEND_URL`).

### Format / lint / test

```bash
npm run format          # Prettier
npm run lint            # ESLint (custom rules included)
npm run test            # Vitest
npm run build           # production build into ./build
```

CI runs all of these on every PR.

## All-in-one Docker stack

For an experience closer to production, use the Compose file under `docker/`:

```bash
cd docker
docker compose up --build
```

It starts:

- The backend image, talking to Fuseki at `host.docker.internal:3030` (so you still need a Fuseki running on the host).
- The frontend image, served via nginx and configured against `/api`.
- An nginx gateway exposed at `http://localhost:3000`.

This is the easiest way to test the path-rewriting reverse proxy locally.

## Verifying the loop works

End-to-end smoke test:

1. Start Fuseki, backend, and frontend.
2. Open `http://localhost:1407`.
3. Create a dataset, import a small Turtle file (one class, one attribute), confirm the diagram renders.
4. Open Swagger UI at `http://localhost:8080/swagger-ui.html` and verify your dataset is visible there.

If all three steps work, the dev loop is fine. If not, see [Run and debug](./run-and-debug).
