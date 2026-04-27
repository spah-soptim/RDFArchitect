---
title: Datasets and Graphs
sidebar_position: 4
---

# Datasets and Graphs

Everything you edit in RDFArchitect lives inside a **graph**, and every graph belongs to a **dataset**. This page is the reference for both.

## The dataset menu

The dataset selector lives in the top toolbar. From there you can:

- **Switch** to another dataset.
- **Create** a new dataset.
- **Delete** an existing dataset (with a confirmation dialog — this also deletes all its graphs and snapshots).
- **Manage namespaces** for the active dataset (see [Prefixes](./prefixes)).
- **Toggle read-only** mode for the dataset.

The currently active dataset is shown next to its name. Every editor action implicitly targets it.

## Dataset settings

Each dataset carries a small amount of metadata:

| Setting | Effect |
| ------- | ------ |
| **Name** | The dataset identifier. Used in URLs and exports. |
| **Read-only** | When set, all editing actions are disabled across all graphs in the dataset. |
| **Namespace prefixes** | Per-dataset overrides on top of the global default prefix list. |

Other than those, datasets are simple containers — there is no quota, no per-dataset role assignment, and no scheduling logic at the application level. Access control is handled by your reverse proxy (see [Access control](/admin-guide/access-control)).

## Graphs in a dataset

Within a dataset you can have any number of graphs. Each graph is independent: importing into one graph never touches another. Common patterns are:

- **One graph per profile.** For CGMES this often means one graph per profile (`EquipmentProfile`, `TopologyProfile`, `SteadyStateHypothesis`, `SVProfile`, …).
- **One graph per version.** Keep `cim16-equipment` and `cim17-equipment` side by side when planning a migration.
- **Working / blessed pair.** A live working graph plus a stable, snapshot-protected baseline.

## The graph menu

In the toolbar's graph selector you can:

- **Switch** the active graph.
- **Create** a new (empty) graph.
- **Delete** a graph. The changelog and snapshots associated with it are removed too.
- **Mark a graph read-only** at any time.
- **Take a snapshot** (see [Snapshots and sharing](./snapshots-and-sharing)).

The active graph is what every action — import, export, edit, comparison, SHACL — applies to.

## Read-only graphs

A graph can be marked read-only independently of its parent dataset. Read-only graphs:

- Show every UI screen, but with editing controls hidden or disabled.
- Cannot accept imports or migrations.
- Can still be exported, compared, and snapshotted.
- Show a clear "read-only" badge in the toolbar.

Read-only is the right state for a "blessed" reference profile that everyone reads but nobody mutates. See [Read-only mode](./readonly-mode) for the full list of consequences.

## Storage location

Internally a dataset corresponds to a Fuseki dataset and graphs are RDF named graphs inside it. Snapshots are stored as additional named graphs under a reserved prefix. Administrators who want to back up or migrate the data can do so at the Fuseki level — see the [Backups](/admin-guide/backups) chapter.

## Limits

There is no hard limit on the number of datasets or graphs. Practical limits are governed by:

- Browser memory when rendering very large diagrams.
- Triple-store performance for very large graphs (millions of triples).

If your model exceeds those limits, split it into multiple graphs along package boundaries.
