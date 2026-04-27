---
title: Import and Export
sidebar_position: 5
---

# Import and Export

RDFArchitect speaks the standard RDF serializations and a clearly defined SHACL convention. This page describes what is supported, what gets recognised, and what to expect on round-trips.

## Importing a schema

Open **Import** from the toolbar.

![Import schema dialog](/img/screenshots/import-schema.png)

The dialog has three inputs:

| Field | Description |
| ----- | ----------- |
| **Dataset** | The destination dataset. Defaults to the active one. |
| **Graph** | Either an existing graph (overwritten) or a new graph name. |
| **File** | The serialized RDF/SHACL file to import. |

### Supported serializations

| Format | Extension | Notes |
| ------ | --------- | ----- |
| Turtle | `.ttl` | Recommended default. |
| RDF/XML | `.rdf`, `.xml` | Works for the standard CIM XMI/RDF exports. |
| N-Triples | `.nt` | |
| N-Quads | `.nq` | Quads are preserved if the file contains named graphs. |
| TriG | `.trig` | |
| JSON-LD | `.jsonld`, `.json` | |

The serialization is detected from the file extension.

### What gets recognised

During import RDFArchitect classifies triples into three buckets:

1. **Schema content** — `rdfs:Class`, `rdf:Property`, `cims:Package`, enum entries, attribute and association declarations, `rdfs:subClassOf`, `rdfs:label`, `rdfs:comment`, `cims:dataType`, `cims:multiplicity`, `cims:stereotype`, fixed values, etc. These become the editable model.
2. **SHACL shapes** — anything reached from a `sh:NodeShape` or `sh:PropertyShape`. These become the **custom shapes** for the graph.
3. **Other triples** — preserved verbatim. RDFArchitect never silently drops triples it doesn't understand; they round-trip on export.

### Layout information

If the imported file contains diagram-layout vocabulary (the `arch:` namespace produced by RDFArchitect itself), it is used to position classes. Otherwise the diagram engine computes a layout automatically the first time you open a package.

## Replacing vs. creating a graph

- **Importing into a new graph name** creates the graph and fills it.
- **Importing into an existing graph** *replaces* its contents. The previous state is captured in the changelog and is therefore restorable, but the working state is fully overwritten.

Importing a SHACL-only file into an existing graph adds those shapes alongside the model — this is the recommended way to add hand-written constraints to a schema you already imported from a different source.

## Exporting a schema

Open **Export** from the toolbar.

You can choose:

- **What to include** — schema only, SHACL only, or both.
- **The serialization** — Turtle is the default and recommended for human-readable diffs.
- **Whether to include diagram layout** — exports the `arch:` layout vocabulary so a re-import comes back with the same visual positioning.

The first resource in an export is always the `Ontology` declaration (with its prefixes), so re-imports are deterministic.

## SHACL export

The SHACL view also has its own export action that emits only the constraint shapes (generated, custom, or both, depending on what you select). See [SHACL](./shacl) for details.

## Exporting all graphs at once

Use the dataset menu's **Export all graphs** action to dump every graph as a single bundle. This is the right command for backups and dataset transfer between installations. Administrators may prefer to back up at the Fuseki level instead — see [Backups](/admin-guide/backups).

## Round-trip guarantees

RDFArchitect aims for a **lossless round-trip**: import → save → export should produce semantically identical RDF. Specifically:

- All triples are preserved, including ones the editor doesn't display.
- Prefixes from the imported file are merged into the dataset's prefix list.
- Diagram layout is preserved when you opt to include it.
- Custom SHACL shapes are preserved unchanged.

Generated SHACL shapes are *not* persisted — they are computed on demand from the model and emitted at export time, so they always reflect the current schema.

## What does not survive

- **Comments inside the source file** (`# this is a comment`). Triple-store storage does not preserve them.
- **Specific blank-node identifiers**. Blank nodes are renamed deterministically on import.
- **Non-canonical literal forms.** For example, `"true"^^xsd:boolean` and `true` collapse to the same value.

If you need exact byte-for-byte preservation, keep the source file in a separate version-control repository in addition to RDFArchitect.
