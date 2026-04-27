---
title: Configuration
sidebar_position: 3
---

# Configuration

A complete reference for the settings that matter.

## Backend configuration

Spring Boot config files live under `backend/src/main/resources` (and can be overridden via environment variables, command-line flags, or external config files).

### `application.yml` — server defaults

| Property | Default | Notes |
| -------- | ------- | ----- |
| `spring.application.name` | `RDFArchitect` | Used in logs and metrics. |
| `spring.servlet.multipart.max-file-size` | `50MB` | Max import file size. |
| `spring.servlet.multipart.max-request-size` | `50MB` | Max request body. |
| `server.tomcat.max-http-form-post-size` | `50MB` | Tomcat-level limit. |
| `server.tomcat.max-swallow-size` | `50MB` | Match the above. |
| `server.tomcat.max-part-count` | `100` | |
| `server.servlet.session.cookie.name` | `RDFA_SESSION_ID` | |
| `server.servlet.session.cookie.same-site` | `lax` | Set to `strict` for stricter deployments. |
| `server.servlet.session.cookie.secure` | `false` | **Set to `true` in production.** |
| `server.servlet.session.cookie.http-only` | `false` | **Set to `true` in production.** |
| `server.servlet.session.timeout` | `365d` | |

### `application-database.yml`

| Property | Default | Notes |
| -------- | ------- | ----- |
| `database.databaseType` | `http` | `http` (Fuseki), `file`, or `inmemory`. |
| `database.defaultDataset` | `default` | Name of the dataset created on first start. |
| `database.http.endpoint` | `http://localhost:3030` | Fuseki URL. **Required in production.** |
| `database.file.endpoint` | `C:/fileDatabase` | Path used when `databaseType=file`. |
| `database.file.dataType` | `trig` | `trig` or `n-quads`. |

### `application-frontend.yml`

| Property | Default | Notes |
| -------- | ------- | ----- |
| `frontend.url` | `http://localhost:1407` | Public frontend origin. Used in CORS and link generation. |
| `frontend.accessRoute` | `/api` | Path under which the backend is served by the gateway. |

### `application-graph.yml`

Graph-level constants, mostly fine at defaults. Adjust only if you have an organisation-wide IRI scheme that diverges from the CIM defaults.

### `application-rendering.yml`

Diagram engine defaults — initial layout spacing, node sizes. Almost never need overriding in production.

## Environment variable overrides

Spring's relaxed binding lets every property be set via an environment variable:

```
DATABASE_HTTP_ENDPOINT=http://fuseki:3030
FRONTEND_URL=https://rdfarchitect.example.com
SERVER_SERVLET_SESSION_COOKIE_SECURE=true
SERVER_SERVLET_SESSION_COOKIE_HTTP_ONLY=true
SPRING_PROFILES_ACTIVE=production
```

Any of these can also live in an external file, supplied via:

```
java -jar app.jar --spring.config.additional-location=file:/etc/rdfarchitect/
```

## Frontend runtime config

The frontend is a static SPA with a small set of placeholders that are rewritten at container start by `frontend/docker-entrypoint.sh`:

| Variable | Purpose |
| -------- | ------- |
| `PUBLIC_BACKEND_URL` | Where the frontend should call the backend. Usually `/api` when there's a gateway. |
| `PUBLIC_APP_VERSION` | Shown in the about dialog. |
| `PUBLIC_COMMIT_SHA` | Shown in the about dialog. |
| `PUBLIC_REPOSITORY_URL` | Link target for "View source". |

These are environment variables on the **frontend container**. They are set in the Compose / orchestrator config, not in the backend.

## Logging

Backend logs are configured via Log4j2 (`backend/src/main/resources/log4j2.yaml`). Default output: structured JSON to stdout, suitable for shipping to ELK / Loki / Cloud Logging.

Override the level for a specific package via env var:

```
LOGGING_LEVEL_ORG_RDFARCHITECT=DEBUG
LOGGING_LEVEL_ORG_APACHE_JENA=INFO
```

## CORS

CORS is auto-configured from `frontend.url`. If you serve the frontend at multiple origins (e.g. www and apex), list them all comma-separated:

```
FRONTEND_URL=https://rdfarchitect.example.com,https://www.rdfarchitect.example.com
```

## Common production settings, all in one place

```yaml
# /etc/rdfarchitect/application.yml
database:
  databaseType: http
  http:
    endpoint: http://fuseki:3030
  defaultDataset: production

frontend:
  url: https://rdfarchitect.example.com
  accessRoute: /api

server:
  servlet:
    session:
      cookie:
        same-site: strict
        secure: true
        http-only: true

logging:
  level:
    org.rdfarchitect: INFO
    org.apache.jena: WARN
```
