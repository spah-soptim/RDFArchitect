---
title: User Guide Overview
sidebar_position: 1
---

# User Guide

This guide walks through everything you can do with RDFArchitect from a browser, in roughly the order you'll encounter the features.

## Reading order

1. **[Concepts](./concepts)** — datasets, graphs, packages, classes, properties, namespaces. Start here if you've never used the tool.
2. **[Getting started](./getting-started)** — open the application, create your first dataset, import a schema.
3. **[Datasets and graphs](./datasets-and-graphs)** — how schemas are organised on disk and in the UI.
4. **[Import & export](./import-export)** — supported formats, what gets imported, what doesn't.
5. **[Editing classes](./editing-classes)** — labels, attributes, associations, enumerations, comments, inheritance.
6. **[Packages and diagrams](./packages-and-diagrams)** — structure your model with packages and explore it as a UML-style diagram.
7. **[SHACL](./shacl)** — generated constraints, custom shapes, and the inspection views.
8. **[Changelog and undo](./changelog-and-undo)** — review history, restore earlier states.
9. **[Snapshots and sharing](./snapshots-and-sharing)** — produce browsable handoff artefacts.
10. **[Comparing schemas](./comparing-schemas)** — diff two graphs or files.
11. **[Schema migration](./migration-wizard)** — the five-step migration wizard.
12. **[Prefixes](./prefixes)** — manage namespace prefixes globally and per dataset.
13. **[Read-only mode](./readonly-mode)** — what changes when a graph is locked.

## What this guide assumes

- You can open a browser and click links.
- You have access to a running RDFArchitect instance (your administrator should have given you a URL such as `https://rdfarchitect.example.com`).
- You know roughly what a CIM schema is. If you don't, see the [CIM/CGMES mapping](/reference/cim-mapping) page first.

You **do not** need to know SPARQL, OWL, Turtle syntax, or the internal structure of Jena to use RDFArchitect. Knowing those things lets you do more, but they are not prerequisites.

## What this guide does not cover

- Building or deploying RDFArchitect yourself — see the [Administration](/admin-guide/installation) section.
- Contributing code — see the [Developer Guide](/developer-guide/overview).
- The REST API surface — when the backend is running, Swagger UI is available at `/swagger-ui.html` on the backend host.
