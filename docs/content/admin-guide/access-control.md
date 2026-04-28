---
title: Access Control
sidebar_position: 6
---

# Access Control

RDFArchitect 1.0.0 has no built-in authentication. The common deployment pattern is to front it with an SSO-capable reverse proxy:

- **OAuth2 Proxy** in front of the nginx gateway, authenticating against your organisation's IdP.
- **Traefik with forward-auth**, if Traefik is already in the stack.
- **Kubernetes Ingress** with an OIDC sidecar (e.g. `oauth2-proxy` as a sidecar container).

The session cookie produced by the backend is called `RDFA_SESSION_ID` and is scoped to the application origin. If you run multiple instances on the same domain, either use different paths or rename the cookie via `server.servlet.session.cookie.name`.

## Snapshot links

Snapshot URLs are capability tokens: whoever has the URL can view. Treat them as sensitive, especially for profiles that have not been publicly released. If you need to revoke a snapshot, delete its Fuseki dataset directly (the name is derivable from the token — `snapshot-<base64>` is the convention).
