---
title: Run and Debug
sidebar_position: 4
---

# Run and Debug

Practical advice for the day-to-day dev loop.

## Running the two services side by side

Open two terminals.

**Terminal 1 — backend:**

```bash
cd backend
mvn spring-boot:run
```

**Terminal 2 — frontend:**

```bash
cd frontend
npm run dev
```

Open `http://localhost:1407`. Backend Swagger UI is at `http://localhost:8080/swagger-ui.html`.

## Hot reload caveats

- **Backend.** Spring DevTools restarts on classpath changes. If you edit a resource bundle (`application*.yml`, SPARQL templates), you may need a `mvn compile` to refresh the classpath. Big config changes usually warrant a full restart anyway.
- **Frontend.** Vite's HMR works for `.svelte`, `.js`, and `.ts`. Changes to `vite.config.js`, `svelte.config.js`, or installed packages require a dev-server restart. Custom ESLint rules run during `npm run lint`, not at HMR time.

## Backend logging

Log4j2 config lives at `backend/src/main/resources/log4j2.yaml`. Bump a package to debug temporarily by overriding via the command line:

```bash
mvn spring-boot:run -Dspring-boot.run.arguments=--logging.level.org.rdfarchitect=DEBUG
```

Spring's structured logger emits per-request audit entries — these are the spine of the changelog feature, not just for ops.

## Frontend debugging

- The browser devtools React-DevTools extension also surfaces Svelte components.
- `console.log` survives HMR until the next manual refresh.
- For state machines built on the runes-based primitives, breakpointing inside `$effect` blocks lets you see the dependency tree.

## Common first-run issues

| Symptom | Likely cause | Fix |
| ------- | ------------ | --- |
| "Failed to fetch" in browser console | Backend not running on `:8080` or CORS misconfigured. | Start backend; check `frontend.url` in `application-frontend.yml` matches the dev origin. |
| Backend errors on startup with "Connection refused" to `:3030` | No Fuseki running. | Start a Fuseki container (see [Setup](./setup)) or switch `databaseType` to `file`. |
| Diagram never renders | A class with malformed multiplicity or missing package. | Check the browser console; the rendering wrapper logs when it skips a class. |
| `npm run dev` fails with module-not-found | Stale `node_modules` after a branch switch. | `npm run clean-install`. |
| `mvn verify` fails on Spotless | Source files not formatted. | `mvn spotless:apply`. |
| `mvn verify` fails on Checkstyle | Style violation. | Run `mvn -Plint -DskipTests verify` to see the report; fix the offence locally. |

## IDE setup

### IntelliJ IDEA

- Import as a Maven project from `backend/pom.xml`.
- Enable annotation processors (Lombok, MapStruct).
- The included `.idea/codeStyle/Project.xml` and `.editorconfig` cover formatting; turn off "Reformat on save" globally and rely on Spotless.

### VS Code

- Recommended extensions: Svelte, Tailwind CSS IntelliSense, ESLint, Prettier.
- Set workspace root to `frontend/` for best linting performance.

## Talking to the running backend

Quick smoke calls:

```bash
curl http://localhost:8080/api/datasets
curl http://localhost:8080/api/datasets/default/graphs
```

Or open Swagger UI at `http://localhost:8080/swagger-ui.html` to browse and try every endpoint interactively.

## Resetting state

- **Reset Fuseki**: stop the container and start it again. The `--rm` flag in the recommended docker run wipes data on stop, which is what you want during dev.
- **Reset the frontend dev origin cache**: `npm run clean-install` from `frontend/`.
- **Reset Maven local cache for this project only**: delete `backend/target/` and re-run `mvn verify`.
