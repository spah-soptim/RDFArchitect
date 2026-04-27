---
title: Installation
sidebar_position: 2
---

# Installation

Three supported paths: **Docker Compose**, **container orchestrator (Kubernetes etc.)**, or **bare-metal**.

## Prerequisites

- An Apache Jena Fuseki instance (or a writable file path if you want to use the file backend).
- A reverse proxy in front for TLS and authentication.
- A persistent volume for Fuseki data.

## Path 1: Docker Compose (single host)

The `docker/docker-compose.yaml` file in the repository is the starting point. For a local trial:

```bash
git clone https://github.com/SOPTIM/RDFArchitect.git
cd RDFArchitect/docker
docker compose up --build
```

The stack listens on `http://localhost:3000`. The Compose file expects a Fuseki server reachable at `host.docker.internal:3030`, so start Fuseki separately:

```bash
docker run -d --name fuseki \
  -p 3030:3030 \
  -e ADMIN_PASSWORD=changeme \
  -v $(pwd)/fuseki-data:/fuseki/databases \
  stain/jena-fuseki:5
```

For a real deployment, copy the file and adapt:

- Pin image tags to specific versions (e.g. `ghcr.io/soptim/rdfarchitect-backend:v1.0.0`).
- Add a Fuseki service to the same Compose file with a named volume for `/fuseki/databases`.
- Add a reverse proxy (Traefik, Caddy, or nginx) for TLS and auth.
- Set `PUBLIC_BACKEND_URL` on the frontend to the public path (`/api`).

### Example production compose

```yaml
services:
  fuseki:
    image: stain/jena-fuseki:5
    volumes:
      - fuseki-data:/fuseki/databases
    environment:
      - ADMIN_PASSWORD_FILE=/run/secrets/fuseki_admin
    secrets:
      - fuseki_admin
    restart: unless-stopped

  backend:
    image: ghcr.io/soptim/rdfarchitect-backend:v1.0.0
    environment:
      DATABASE_DATABASETYPE: http
      DATABASE_HTTP_ENDPOINT: http://fuseki:3030
      DATABASE_DEFAULTDATASET: default
      FRONTEND_URL: https://rdfarchitect.example.com
    depends_on: [fuseki]

  frontend:
    image: ghcr.io/soptim/rdfarchitect-frontend:v1.0.0
    environment:
      PUBLIC_BACKEND_URL: /api
    depends_on: [backend]

  gateway:
    image: traefik:v3
    # …TLS + auth config…
    ports: ["443:443", "80:80"]

volumes:
  fuseki-data:

secrets:
  fuseki_admin:
    file: ./secrets/fuseki_admin
```

## Path 2: Kubernetes / orchestrator

Translate the Compose layout into:

- A Deployment + Service + PVC for Fuseki.
- A Deployment + Service for the backend.
- A Deployment + Service for the frontend.
- An Ingress (or your usual gateway) for TLS, auth, and routing.

Helm charts are not provided in 1.0; the repo welcomes a contribution.

## Path 3: Bare-metal

Build artefacts:

```bash
# Backend
cd backend
mvn -B verify
# Result: target/RDFArchitect-*.jar

# Frontend
cd ../frontend
npm run clean-install
npm run build
# Result: build/  (static site)
```

Run them:

```bash
# Backend (systemd unit recommended)
java -jar backend/target/RDFArchitect-*.jar \
  --spring.config.additional-location=/etc/rdfarchitect/

# Frontend (nginx serving build/ + reverse-proxy /api → backend)
```

Provide config files at `/etc/rdfarchitect/application.yml` (and optionally `application-database.yml` etc.) overriding the defaults. See [Configuration](./configuration).

## Smoke testing the install

After install, verify:

1. `https://rdfarchitect.example.com/` returns the frontend.
2. `https://rdfarchitect.example.com/api/datasets` returns JSON (and authentication is enforced if you configured it).
3. `https://rdfarchitect.example.com/api/swagger-ui.html` shows the API explorer.
4. Creating a dataset via the UI succeeds and persists across a backend restart.

If any step fails, see [Troubleshooting](./troubleshooting).

## Production hardening checklist

- [ ] TLS terminated at the reverse proxy, HSTS enabled.
- [ ] Authentication required for all paths.
- [ ] Fuseki on a private network only.
- [ ] Fuseki admin password set, not the default.
- [ ] Backups scheduled (see [Backups](./backups)).
- [ ] Logs shipped to your aggregator.
- [ ] Resource limits on backend and Fuseki containers.
- [ ] Image tags pinned, not `:latest`.
- [ ] Session cookie set to `Secure`, `HttpOnly`, `SameSite=Lax` (override the dev defaults — see [Configuration](./configuration)).
