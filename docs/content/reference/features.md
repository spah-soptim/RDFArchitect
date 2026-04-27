---
title: Feature Checklist
sidebar_position: 1
---

# Feature Checklist

A quick map of what's in the box, organised by who tends to use what.

## Power-system / electrical engineers

| Feature | Where in the UI |
| ------- | --------------- |
| Browse a CIM/CGMES schema as a UML-style diagram | Editor → diagram canvas |
| Drill into a class's attributes, associations, and inheritance | Editor → class editor |
| Inspect generated and custom SHACL constraints on a class | Class editor → SHACL tab |
| Compare a working schema against a snapshot | Toolbar → Compare |
| Receive a snapshot link, browse without an account | Snapshot URL |
| View change history of a graph | Toolbar → Changelog |

## Information modelers / data architects

| Feature | Where in the UI |
| ------- | --------------- |
| Create classes, attributes, associations, enumerations, packages | Class editor / package editor / new-class dialog |
| Single-inheritance modelling | Class editor → General tab → Parent class |
| Multiplicity, datatypes, fixed values, stereotypes | Class editor → Attributes tab |
| Custom SHACL upload | SHACL menu → Upload |
| Auto-generated SHACL exposed for review and export | SHACL menu → Full SHACL view |
| Per-graph and per-dataset namespace management | Dataset menu → Manage namespaces |
| Graph import (Turtle, RDF/XML, N-Triples, N-Quads, TriG, JSON-LD) | Toolbar → Import |
| Graph export with optional layout & SHACL | Toolbar → Export |
| Schema comparison with package/class/property granularity | Toolbar → Compare |
| Five-step migration wizard producing a SPARQL Update script | Toolbar → Migrate |

## Project managers / product owners

| Feature | Where in the UI |
| ------- | --------------- |
| Read-only browse of any dataset / graph / snapshot | Read-only badge in the toolbar |
| Stable, shareable snapshot links | Graph menu → Snapshot |
| Per-graph changelog with restore | Toolbar → Changelog |
| Comparison of two snapshots / files | Toolbar → Compare |

## Developers

| Feature | Where |
| ------- | ----- |
| REST API (Swagger UI) | `/swagger-ui.html` on the backend |
| Open source under Apache 2.0 | [github.com/SOPTIM/RDFArchitect](https://github.com/SOPTIM/RDFArchitect) |
| Hexagonal backend architecture (Java 25, Spring Boot 4, Jena 5) | [Backend architecture](/developer-guide/backend-architecture) |
| SvelteKit frontend (Svelte 5 runes, Vite 7, Tailwind 4) | [Frontend architecture](/developer-guide/frontend-architecture) |
| Containerised deploy (Docker, GHCR images) | `docker/docker-compose.yaml` |
| Tag-driven releases | [CI and releases](/developer-guide/ci-and-releases) |

## Administrators

| Feature | Where |
| ------- | ----- |
| Pluggable storage backends (Fuseki HTTP, file, in-memory) | `application-database.yml` |
| Read-only mode at dataset, graph, or application level | UI toggles + `frontend.url` config |
| Audit-grade changelog | Per-graph, persisted in Fuseki |
| Reverse-proxy-friendly auth integration (`X-Authenticated-User`) | [Access control](/admin-guide/access-control) |

## Where in the UI: quick reference

| Action | Path |
| ------ | ---- |
| Welcome | `/` |
| Editor | `/mainpage` |
| Changelog | `/changelog` |
| Compare | `/compare` |
| Migration wizard | `/migrate` |
| SHACL view | `/shacl` |
| Prefix manager | `/prefixes` |

## What's not (yet) in the box

See [Limitations](./limitations).
