---
title: Overview
sidebar_position: 1
---

# Administrator's Guide

Topics for the person operating RDFArchitect for a team: configuration, triple store, backups, security, and common operational tasks.

## Architecture at a glance

RDFArchitect consists of three runtime components plus a triple store:

- **Frontend** — a SvelteKit single-page application served as static files behind an nginx process.
- **Backend** — a Spring Boot REST service. Holds no long-term state of its own; every persistent change is written through to the triple store.
- **Gateway** (optional) — an nginx reverse proxy that routes `/api/*` to the backend and everything else to the frontend, so that both can be served from a single origin.
- **Triple store** — **Apache Jena Fuseki** (recommended). Stores all datasets, graphs, SHACL, and snapshots.

Every RDFArchitect dataset is a Fuseki dataset. Every RDFArchitect graph is a named graph inside that Fuseki dataset. Every SHACL document is stored in a dedicated named graph next to its schema graph. Every snapshot is stored as its own dataset with a name derived from the base64 snapshot token.

This mapping is deliberately transparent: you can inspect RDFArchitect's state with any Fuseki client, run arbitrary SPARQL against it, back it up with Fuseki's own tools, and restore it the same way.

## Where to go next

- [Installation](./installation) — how to deploy.
- [Configuration](./configuration) — settings reference.
- [Apache Jena Fuseki](./fuseki) — triple-store guidance.
- [Backups](./backups) — backup and restore.
- [Security](./security) — trusted-network and reverse-proxy guidance.
- [Monitoring](./monitoring) — health and metrics.
- [Upgrading](./upgrading) — version-to-version notes.
- [Troubleshooting](./troubleshooting) — operational issues.
