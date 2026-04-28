# RDFArchitect — Administrator's Guide

Topics for the person operating RDFArchitect for a team: configuration, triple store, backups, access control, and common operational tasks.

---

## Architecture at a glance

RDFArchitect consists of three runtime components plus a triple store:

- **Frontend** — a SvelteKit single-page application served as static files behind an nginx process.
- **Backend** — a Spring Boot REST service. Holds no long-term state of its own; every persistent change is written through to the triple store.
- **Gateway** (optional) — an nginx reverse proxy that routes `/api/*` to the backend and everything else to the frontend, so that both can be served from a single origin.
- **Triple store** — **Apache Jena Fuseki** (recommended). Stores all datasets, graphs, SHACL, and snapshots.

Every RDFArchitect dataset is a Fuseki dataset. Every RDFArchitect graph is a named graph inside that Fuseki dataset. Every SHACL document is stored in a dedicated named graph next to its schema graph. Every snapshot is stored as its own dataset with a name derived from the base64 snapshot token.

This mapping is deliberately transparent: you can inspect RDFArchitect's state with any Fuseki client, run arbitrary SPARQL against it, back it up with Fuseki's own tools, and restore it the same way.

---

## Configuration

The authoritative configuration is `backend/src/main/resources/application.yml`. Every value can be overridden at runtime with environment variables using Spring Boot's standard mapping (dots → underscores, uppercase). The compose file sets `DATABASE_DATABASETYPE`, `DATABASE_DEFAULTDATASET`, and `DATABASE_HTTP_ENDPOINT` this way.

### Commonly adjusted settings

| Purpose                            | Property                                | Default                      |
| ---------------------------------- | --------------------------------------- | ---------------------------- |
| Triple store endpoint              | `database.http.endpoint`                | `http://localhost:3030`      |
| Default dataset name               | `database.defaultDataset`               | `default`                    |
| Frontend URL (CORS allow-list)     | `frontend.url`                          | `http://localhost:1407`      |
| API base path                      | `frontend.accessRoute`                  | `/api`                       |
| Max uploaded schema size           | `spring.servlet.multipart.max-file-size`| `50MB`                       |
| History depth per graph            | `graph.maxVersions`                     | `256`                        |
| Diagram renderer                   | `rendering.renderer`                    | `svelteflow` (or `mermaid`)  |
| Session cookie name                | `server.servlet.session.cookie.name`    | `RDFA_SESSION_ID`            |
| Session cookie `secure` flag       | `server.servlet.session.cookie.secure`  | `false` (set to `true` in production) |

### File-based storage (no Fuseki)

For small or offline use you can set `database.databaseType=file`. In that mode the backend stores data as TriG or N-Quads files under `database.file.endpoint`. This is not recommended for anything beyond quick experiments — there is no concurrent access protection and no server-side SPARQL engine.

---

## Triple store: Fuseki

### Recommended deployment

- A single Fuseki instance, TDB2-backed.
- One Fuseki dataset per RDFArchitect dataset (typically: `default`, plus one per active snapshot).
- Persistent volume under `/fuseki` with enough space for schemas, SHACL, version history, and snapshots (a few hundred MB per active CGMES release is typical).

### Access control between RDFArchitect and Fuseki

RDFArchitect talks to Fuseki over HTTP using the SPARQL 1.1 Protocol (SELECT/UPDATE) plus the Graph Store Protocol (GSP). If Fuseki is on the same private network as the backend, no authentication is usually configured.

If you put Fuseki behind basic auth or an HTTP-level auth proxy, RDFArchitect does not currently send credentials. Either keep Fuseki on a trusted network, or place both behind the same authenticating proxy so the request path from the browser is the only one that needs to authenticate.

### Read-only users and snapshots

RDFArchitect's read-only model is per dataset, enforced by the backend. It is **not** a Fuseki-level protection — a user with direct Fuseki access can still modify any dataset. For stronger guarantees, either:

- Keep Fuseki on a private network reachable only by the backend.
- Run two Fuseki instances (read-write for editing, read-only for snapshots) and route reads and writes accordingly. This is not out-of-the-box; it requires a routing layer in front of Fuseki.

---

## Backups

RDFArchitect does not hold long-term state. Backup = Fuseki backup.

### Recommended routine

Fuseki has a built-in backup API:

```
curl -XPOST http://<fuseki-host>:3030/$/backup/<dataset>
```

This produces a gzipped N-Quads dump under Fuseki's `backups/` directory. A cron job that calls this once a day for each active dataset and rotates the last N dumps covers the standard disaster-recovery case.

### Restoring

A dump produced by `/$/backup/<dataset>` is a standard N-Quads file and can be loaded back via Fuseki's data upload endpoint:

```
curl -XPOST -H 'Content-Type: application/n-quads' \
     --data-binary @backup.nq.gz \
     -H 'Content-Encoding: gzip' \
     http://<fuseki-host>:3030/<dataset>/data
```

### What to back up

- **Every dataset** RDFArchitect knows about. In practice: `default`, plus every snapshot dataset.
- **Namespace and configuration state** is stored inside each dataset, so no separate backup is needed for it.

### What does not need to be backed up

- The backend itself — stateless, can be rebuilt from the image.
- The frontend — stateless, can be rebuilt from the image.
- The gateway configuration — in version control.

---

## Access control

RDFArchitect 1.0.0 has no built-in authentication. The common deployment pattern is to front it with an SSO-capable reverse proxy:

- **OAuth2 Proxy** in front of the nginx gateway, authenticating against your organisation's IdP.
- **Traefik with forward-auth**, if Traefik is already in the stack.
- **Kubernetes Ingress** with an OIDC sidecar (e.g. `oauth2-proxy` as a sidecar container).

The session cookie produced by the backend is called `RDFA_SESSION_ID` and is scoped to the application origin. If you run multiple instances on the same domain, either use different paths or rename the cookie via `server.servlet.session.cookie.name`.

### Snapshot links

Snapshot URLs are capability tokens: whoever has the URL can view. Treat them as sensitive, especially for profiles that have not been publicly released. If you need to revoke a snapshot, delete its Fuseki dataset directly (the name is derivable from the token — `snapshot-<base64>` is the convention).

---

## Monitoring

The backend exposes the standard Spring Boot Actuator endpoints at `/actuator/*`. At minimum, `health` and `info` are suitable for a container platform's liveness/readiness checks. Prometheus-style metrics can be enabled by adding `micrometer-registry-prometheus` to the backend dependencies — this is not shipped by default.

On the Fuseki side, the `/$/metrics` endpoint provides Prometheus metrics out of the box. Scraping both sides is the recommended monitoring baseline.

---

## Upgrading

RDFArchitect follows semantic versioning (see `CHANGELOG.md`). For minor and patch releases, upgrading is typically a straight image swap:

```
docker compose pull
docker compose up -d --build
```

For major releases, review the changelog for breaking changes before upgrading. The 0.15.0 release, for example, switched the default diagram renderer from Mermaid to SvelteFlow and required a one-time layout regeneration on existing graphs.

### Data compatibility

RDFS and SHACL artefacts stored in the triple store are standard RDF; they are forward and backward compatible across RDFArchitect versions.

### Frontend caching

After an upgrade, users may see stale frontend assets. A hard refresh (Ctrl+Shift+R) is usually sufficient; for public deployments, configure the nginx gateway to include a content-hash in static asset URLs to avoid the issue entirely.

---

## Scaling

RDFArchitect is designed for small teams (dozens of users) rather than public internet scale. The limiting factor is almost always the triple store:

- One Fuseki instance on modest hardware (4 cores, 8 GB RAM, SSD) comfortably serves a team of 20–30 concurrent modellers on a full set of CGMES + ENTSO-E profiles.
- The backend is stateless and can be scaled horizontally behind a load balancer, with session affinity, if needed. The frontend is static and trivially scalable.
- The triple store itself is not horizontally scalable in the open-source Fuseki. For very large or very busy deployments, a commercial triple store that supports clustering can be dropped in — RDFArchitect only requires SPARQL 1.1 + GSP.

---

## Troubleshooting basics

See [FAQ & troubleshooting](faq.md) for user-facing issues.

For operational issues:

- Backend logs are standard Spring Boot logs (log4j2), on stdout. Every REST call is logged at INFO level with the dataset, graph, and originating URL.
- Frontend build and runtime issues are reported in the browser console and in the SvelteKit server log.
- Triple store issues surface as HTTP errors in the backend log. A `Connection refused` means Fuseki is down or unreachable from the backend; a `403` means Fuseki is refusing the write (often because the dataset is configured as read-only on the Fuseki side).

For anything not covered here, the GitHub repository's [Discussions](https://github.com/SOPTIM/RDFArchitect/discussions) tab is the recommended channel.
