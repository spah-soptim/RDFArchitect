---
title: Administration Overview
sidebar_position: 1
---

# Administration

This part of the docs is for the people who deploy, configure, secure, and back up RDFArchitect.

## What you need to know

RDFArchitect is **two services and a triple store**:

- A **backend** Java/Spring application (default port `8080`).
- A **frontend** static site served by nginx (default port `80`).
- An **Apache Jena Fuseki** triple store (default port `3030`).

In a production deployment you place a reverse proxy in front of the frontend and backend, and Fuseki sits on a private network behind them. Authentication, TLS, and rate limiting are handled at the reverse proxy — RDFArchitect itself has no built-in user management.

## Reading order

1. **[Installation](./installation)** — Docker Compose, bare-metal install, deployment topologies.
2. **[Configuration](./configuration)** — every backend and frontend setting that matters.
3. **[Apache Jena Fuseki](./fuseki)** — provisioning, data location, security.
4. **[Access control](./access-control)** — reverse-proxy auth patterns and identity propagation.
5. **[Backups](./backups)** — what to back up, how to restore.
6. **[Upgrades](./upgrades)** — version-to-version notes and migration between schema versions.
7. **[Scaling](./scaling)** — when and how to scale, what doesn't scale.
8. **[Troubleshooting](./troubleshooting)** — most common operational issues.

## Operating principles

- **Treat Fuseki as the system of record.** RDFArchitect is stateless and depends entirely on Fuseki for persistence.
- **Back up Fuseki, not just RDFArchitect.** A "RDFArchitect backup" without the matching Fuseki state is useless.
- **Don't expose Fuseki publicly.** It has its own admin UI; it is not designed to face the internet.
- **Authentication belongs at the proxy.** Don't try to teach the application about user accounts; let your IDP and proxy do it.

## Reference deployment

The minimal trustworthy production deployment looks like:

```
[Internet]
   │
   ▼
[Reverse proxy (Traefik / nginx / Apache)]
   │   - TLS terminator
   │   - SSO / OIDC / LDAP authenticator
   │   - Injects X-Authenticated-User
   ▼
┌──────────────────────────────────┐
│  Frontend container (nginx)      │
│   serves /  → static SPA         │
│   proxies /api → backend         │
└──────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  Backend container (Java 25)     │
│   :8080                          │
└──────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  Apache Jena Fuseki              │
│   :3030  (private network only)  │
│   persistent volume mounted      │
└──────────────────────────────────┘
```

The local Docker Compose file under `docker/` is a stripped-down version of this — useful for development, not suitable as-is for production.
