---
title: Getting Started
sidebar_position: 3
---

# Getting Started

This page takes you from "I have a URL" to "I'm editing a CIM schema" in about ten minutes.

## 1. Open the application

Browse to the URL your administrator gave you (locally, this is usually [http://localhost:3000](http://localhost:3000) for the Docker setup or [http://localhost:1407](http://localhost:1407) when running the frontend dev server).

The welcome page lists the available datasets and lets you jump into the editor.

![Homepage](/img/screenshots/homepage.png)

## 2. Make sure a dataset exists

A fresh installation starts with a dataset called `default`. If you need a clean workspace, create a new one from the dataset menu — give it a short, descriptive name (e.g. `cim-100`, `entso-e-cgmes-3`) so you can find it quickly later.

> Datasets are cheap. Use one per profile family, customer engagement, or compliance scope.

## 3. Import a schema

Open **Import** from the toolbar. The dialog asks for:

- **Target dataset** — usually the one you just created.
- **Target graph** — pick "create new graph" and give it a name (e.g. `EquipmentProfile`).
- **File** — the RDFS, OWL, or Turtle file you want to load. SHACL shapes inside the file are recognised and stored as custom shapes for that graph.

![Import dialog](/img/screenshots/import-schema.png)

Supported serializations include Turtle (`.ttl`), RDF/XML (`.rdf`, `.xml`), N-Triples (`.nt`), N-Quads (`.nq`), TriG (`.trig`), and JSON-LD (`.jsonld`). The serialization is detected from the file extension.

When the import finishes, the target graph becomes the active graph and the editor opens.

## 4. Browse the model

The editor has three working areas:

1. **Left panel — package & class navigation.** Drill into the package tree, search by name, or reach a class via the global search bar.
2. **Center — diagram view.** A package's classes are rendered as a UML-style diagram. Click a class to focus it, double-click to open it in the class editor.
3. **Right panel — context actions.** Class editor, SHACL inspection, change history.

![Editor overview](/img/screenshots/editor.png)

## 5. Make a change

Try one of the following to confirm the round-trip works:

- Open a class in the class editor and tweak its `rdfs:comment`.
- Add a new attribute (give it a name, datatype, and multiplicity).
- Create a new package from the navigation tree, then add a fresh class to it.

Save the change. A new entry appears in the **changelog**, and the **diagram** redraws automatically.

## 6. Generate or inspect SHACL

From the SHACL menu choose **View generated shapes** to see the constraints RDFArchitect derived from your model. If your import contained custom SHACL, those are visible alongside the generated ones, clearly marked as custom.

![SHACL inspection](/img/screenshots/shacl.png)

## 7. Share your work

Two complementary mechanisms:

- **Export** the graph as Turtle (or RDF/XML, etc.) — useful for handover to other tools.
- **Snapshot** a graph and share the link — useful for review by colleagues who do not need to edit.

Snapshots are a fully browsable, read-only view of the schema. The reviewer needs the URL and any access credentials your administrator has configured at the reverse proxy level.

## What next?

- Read [Editing classes](./editing-classes) for the full editor reference.
- Read [SHACL](./shacl) for generated vs. custom shapes.
- Read [Migration wizard](./migration-wizard) when you need to move data between schema versions.
