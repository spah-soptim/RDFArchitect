# RDFArchitect — User Guide

A practical walkthrough of RDFArchitect for people who work with **CIM/CGMES** and the **ENTSO-E Network Code Profiles**, but who do not want to write code.

RDFArchitect is a browser-based editor for **RDFS schemas with CIM extensions** and their **SHACL constraints**. It is designed around the daily work of the CIM community: importing a profile, inspecting its package and class structure, making controlled changes, validating against SHACL, comparing releases, planning a migration, and sharing the result.

This guide assumes you have the application running. If you don't, see [Installation](installation.md).

---

## Table of Contents

1. [Core concepts in 2 minutes](#1-core-concepts-in-2-minutes)
2. [The workspace](#2-the-workspace)
3. [Getting data into RDFArchitect](#3-getting-data-into-rdfarchitect)
4. [Organising a schema: datasets, graphs, packages](#4-organising-a-schema-datasets-graphs-packages)
5. [Editing classes](#5-editing-classes)
6. [Working with namespaces](#6-working-with-namespaces)
7. [The profile header (ontology metadata)](#7-the-profile-header-ontology-metadata)
8. [SHACL — constraints and validation](#8-shacl--constraints-and-validation)
9. [Reviewing changes — changelog, undo, restore](#9-reviewing-changes--changelog-undo-restore)
10. [Comparing schemas](#10-comparing-schemas)
11. [Schema migration](#11-schema-migration)
12. [Sharing and exporting](#12-sharing-and-exporting)
13. [Read-only mode](#13-read-only-mode)
14. [Search](#14-search)
15. [Tips & keyboard shortcuts](#15-tips--keyboard-shortcuts)

---

## 1. Core concepts in 2 minutes

Before the tour, three terms do a lot of work in RDFArchitect and it is worth fixing them up front.

**Dataset.** A dataset is the outermost container. Think of it as a "workspace" on the underlying triple store. A typical setup has one dataset called `default`, plus one dataset per snapshot that has been shared. Every import, every graph, every change is scoped to exactly one dataset.

**Graph (schema).** Inside a dataset, each **schema** lives in its own named graph. When you import a CGMES profile — say, `EquipmentProfile_v3.0.0.rdf` — that profile becomes *one graph* inside the dataset. You can have many graphs side-by-side in the same dataset (e.g. EQ, TP, SSH, SV profiles of a CGMES release), and you can move between them from the navigation tree on the left.

**Package.** Inside a graph, classes are grouped into UML-style packages (e.g. `Core`, `Wires`, `Generation::Production`). Packages are the first-class organisational unit for the diagram area: when you select a package, the centre canvas draws exactly the classes that belong to it, with associations to classes in other packages shown at the boundary.

The rest of the editor is built on top of those three. The navigation tree reads *"Dataset → Graph → Package → Class"*, and almost every action you trigger is implicitly scoped by what you have selected in that tree.

---

## 2. The workspace

The main editor, reachable from the **Open Editor** button on the homepage or via the top navigation, is split into three panels:

- **Left — navigation tree.** Datasets, graphs inside each dataset, and packages inside each graph. Classes live under their packages and can be opened from here.
- **Centre — diagram canvas.** A UML-style class diagram of the currently selected package: classes with their attributes and associations, inheritance arrows, and links that leave the package. The diagram is rendered with SvelteFlow by default and supports pan, zoom, and auto-layout; a Mermaid rendering is also available.
- **Right — class editor.** When a class is selected, its full definition is available here: label, URI namespace, package, super class, stereotypes, comment, attributes, associations, enum entries, and class-level SHACL.

Across the top there is a **menu bar** (File, Edit, View, Help), a **search bar** for finding classes/attributes/packages across datasets, and indicators for read-only status and unsaved changes.

A colour legend and classic diagram conventions are used throughout: stereotypes like `«enumeration»` or `«Primitive»` appear above the class name, cardinalities are shown on association ends, and inheritance uses a hollow triangle.

---

## 3. Getting data into RDFArchitect

There are three ways content lands in the editor.

**Import a schema (File → Import → Schema (RDFS)).** This is the standard entry point for CGMES profiles and ENTSO-E Network Code Profiles. Pick a dataset (existing or a new name), drop in an `.rdf`, `.ttl`, or `.nt` file, and the contents become a graph inside that dataset. The graph name defaults to a sanitised version of the filename but can be overridden. If the target dataset has been set to read-only, the import is blocked until editing is re-enabled.

Supported input formats are **RDF/XML (.rdf)**, **Turtle (.ttl)**, and **N-Triples (.nt)**. CGMES releases are typically shipped as RDF/XML and can be imported directly.

**Import a SHACL file (File → Import → Constrains (SHACL)).** A separate import path for custom SHACL shapes. These are stored *next to* the schema graph and can later be viewed from the same UI as the generated SHACL (see [SHACL](#8-shacl--constraints-and-validation)).

**Create an empty schema (Edit → New → Schema (RDFS)).** Starts a new, empty graph in the selected dataset. Useful when you want to build a small extension profile from scratch rather than import one.

Once a schema is imported, it becomes the active graph and the navigation tree on the left updates to show its packages.

---

## 4. Organising a schema: datasets, graphs, packages

### Creating a package

With a graph selected, choose **Edit → New → Package**. You provide a label, a URI namespace, and (optionally) a parent package for nesting. Packages can be edited (**Edit → Edit → Package**) and deleted (**Edit → Delete → Package**) from the same menu.

The `default` package is reserved — it represents classes that have not been assigned to any explicit package. It cannot be renamed or deleted.

### External vs. internal packages

A graph usually contains two categories of packages:

- **Internal packages** — defined in the schema itself; fully editable.
- **External packages** — referenced from another imported schema (for example, the `Core` package of CGMES when you are editing an extension profile). They are visible for navigation and class assignment but not editable.

The navigation tree marks these two categories separately, and the editing menus disable editing actions for external packages automatically.

### Creating a class

From **Edit → New → Class** you pick the target dataset, graph, package, and a URI namespace for the new class. The label must be unique within the graph; the editor validates this before allowing save. On save, the class opens immediately in the class editor on the right.

### Deleting

Every destructive action (delete schema, delete dataset, delete package, delete class) goes through a confirmation dialog that states what will be removed. Deletions participate in undo/redo like any other edit (see [Reviewing changes](#9-reviewing-changes--changelog-undo-restore)).

---

## 5. Editing classes

The class editor on the right-hand side is the main surface for modelling work. It is laid out so that everything about a single class is reachable from one scroll, without navigating away.

### What you can edit

- **Label and URI namespace.** The human-readable name and the namespace it lives under. The editor enforces label uniqueness and flags invalid characters inline.
- **Package.** Moves the class between packages in the current graph.
- **Super class.** Sets or clears inheritance. The picker shows all classes from the current graph and any external packages it references.
- **Stereotypes.** CIM uses stereotypes heavily (`«enumeration»`, `«CIMDatatype»`, `«Primitive»`, `«Compound»`, etc.). They are selected from the list of known stereotypes and shown in the diagram above the class name.
- **Comment.** Free-text description, rendered as AsciiDoc in the class editor so that lists, code snippets, and links are formatted sensibly when reading back a profile.
- **Attributes.** Data-typed properties. Each row defines a local name, URI namespace, datatype (from the CIM primitive datatypes or xsd types), cardinality, a fixed value if any, and a comment.
- **Associations.** Links to other classes. You set the target class, role name, multiplicity (`0..1`, `1`, `0..*`, `1..*`), the inverse role where applicable, and a comment.
- **Enum entries.** Present only when the class has the `«enumeration»` stereotype. Each entry has a label, URI, and comment and is ordered in the list.
- **SHACL on a property.** Every attribute and association row has a small icon that opens the property-specific SHACL dialog (see [SHACL](#8-shacl--constraints-and-validation)).
- **UUID.** The internal resource UUID is shown read-only. It is stable across edits and is what RDFArchitect uses internally to refer to the class.

### Validation as you type

The editor does not wait for save to tell you something is wrong. Label collisions, empty required fields, duplicate attribute names, invalid URI components, and SHACL violations caused by the pending changes are all reported inline as you type. The **Save** button stays disabled while there are unresolved issues, and a list of violations is shown above the button.

### Discard or adopt unsaved changes

If you switch classes while there are unsaved edits, RDFArchitect asks whether to save, discard, or **adopt** them. "Adopt" means: carry the pending changes over to the next class where they still apply — useful when you are making the same correction across a family of classes.

---

## 6. Working with namespaces

Namespaces are managed per dataset from **Edit → Manage Namespaces** (or **View Namespaces** when read-only). The dialog lists every prefix/URI pair currently defined for the dataset, lets you add new prefixes, rename them, or remove unused ones.

Prefix uniqueness is enforced: the save button stays disabled while two rows share a prefix, and the offending rows are highlighted. Namespaces used by any resource in the dataset cannot be deleted — the editor flags them as in-use.

Namespaces are also surfaced in every place where a URI is entered (new class dialog, attribute editor, import/export dialogs) so that you rarely need to type the full namespace by hand.

---

## 7. The profile header (ontology metadata)

Every CGMES / ENTSO-E profile has a metadata block that identifies it — title, identified version, keyword, licence, conformsTo, description, and so on. In RDFArchitect this is the **Profile header**, reachable from **Edit → Edit → Profile header** (the menu entry shows **View** or **Add** depending on state).

The header editor is a row-based dialog: each row is a known field (e.g. `dcat:version`, `dcterms:title`, `cim:Ontology.baseUriScheme`) with its value. A list of *known* ontology fields — the ones standardised by ENTSO-E and commonly used in CGMES profiles — can be added in one click from the **Add known fields** sub-dialog, so you don't have to remember the exact predicate names.

On export, the profile header is written as the first resource of the file — matching the convention used by ENTSO-E's official releases — and can optionally be re-generated from schema metadata at export time.

---

## 8. SHACL — constraints and validation

SHACL (Shapes Constraint Language) is how CGMES and ENTSO-E express the data-quality rules that an exchange file must satisfy: "every `ACLineSegment` must have exactly one `length`", "every `Terminal` must reference a `ConductingEquipment`", and so on. RDFArchitect treats SHACL as a first-class citizen.

### Two sources of SHACL

RDFArchitect distinguishes two kinds of shapes and stores them separately:

- **Generated SHACL.** Produced by RDFArchitect from the schema itself. Every class becomes a `NodeShape`; every attribute and association becomes a `PropertyShape` with `sh:path`, `sh:datatype` or `sh:class`, `sh:minCount`, `sh:maxCount`, and `sh:in` (for enums) derived from what you modelled. This set is always in sync with the current state of the graph.
- **Custom SHACL.** Shapes that you import or author separately — typically the official SHACL files that ship with a CGMES or ENTSO-E release. These are preserved byte-for-byte and are *not* regenerated when the schema changes.

When you view SHACL for a graph, both sets are shown and clearly labelled.

### Viewing SHACL at graph level

**View → Constrains (SHACL)** opens the full-view dialog. Two tabs: **Generated** (read-only TTL output) and **Custom** (editable TTL). The custom tab has inline TTL syntax highlighting and validates as you type; the save button stays disabled until the TTL parses.

### Viewing SHACL at class level

In the class editor, every attribute and association row has a SHACL icon. Clicking it opens the **property-specific SHACL dialog** — the subset of both generated and custom shapes that target that exact property on that exact class. This is by far the fastest way to answer the question *"what constraint is enforced on this attribute?"* without leaving the class you are looking at.

A similar dialog exists at the class level to inspect the full `NodeShape` of the selected class.

### Importing custom SHACL

**File → Import → Constrains (SHACL)** uploads a SHACL file into the currently selected graph. Supported formats are the same as for schema import (TTL, RDF/XML, N-Triples); TTL is the default and recommended format.

### Exporting SHACL

**File → Export → Constrains (SHACL)** downloads a SHACL file. The dialog asks which dataset and graph to use, which parts to include (generated, custom, or both), and in which format. TTL is the default.

---

## 9. Reviewing changes — changelog, undo, restore

Every edit you make to a graph is tracked. Three features make this tracking visible and reversible.

### Undo / Redo

Cross-cutting, session-level undo and redo for *every* edit — class, attribute, association, enum entry, package, namespace, ontology — is available from **Edit → Undo** / **Redo** or with **Ctrl+Z** / **Ctrl+Y**. There is no per-entity undo history; undo walks the whole graph's edit stream linearly.

The backend keeps up to 256 versions per graph by default (configurable) and compresses older states as you keep editing.

### Changelog view

**View → Changelog** opens a dedicated page that lists the change history of the currently selected graph: who changed what, what was added, updated, or deleted, and an inline diff for each change. The left pane groups changes by class, the right pane shows the full triple-level detail of the selected change with additions in green and deletions in red.

This is the view to use when reviewing what happened between two editing sessions, or when preparing a release note.

### Restore a previous version

From the changelog you can restore the graph to any earlier point. This creates a new entry at the top of the history (so it is itself undoable) and resets every class, package, and namespace to the state it had at that time. It does not touch the custom SHACL content unless the SHACL was also part of the restored history.

---

## 10. Comparing schemas

**View → Compare Schemas** opens the compare dialog. There are three comparison modes:

- **Stored ↔ Stored.** Compare two graphs already loaded in RDFArchitect — typical case: comparing two CGMES releases you have imported side-by-side, or a working copy against a pristine baseline.
- **Uploaded ↔ Stored.** Compare a file on your disk against a stored graph — useful when checking a proposed new version from an external party against the version you currently have.
- **Uploaded ↔ Uploaded.** Compare two files on disk, no storage needed.

The result is a structured change list grouped by package and then by class, with badges for *added*, *removed*, *changed* at every level (package, class, attribute, association, enum entry, comment). A class change expands to show exactly which attributes or associations moved. Comment changes are whitespace-normalised, so pure reformatting does not pollute the diff.

The compare view is the recommended starting point when planning a migration.

---

## 11. Schema migration

RDFArchitect ships a guided **Schema Migration** workflow — **View → Migrate Schema** — that turns the differences between two schema versions into an executable **SPARQL UPDATE** script. This script, when run against an exchange dataset that conforms to the *source* schema, transforms it into a dataset that conforms to the *target* schema.

Migration runs as a five-step wizard:

**Step 1 — Select Schemas.** Pick the source and the target. Same three modes as compare (stored/stored, uploaded/stored, uploaded/uploaded). RDFArchitect computes the difference and uses it as the starting point for the remaining steps.

**Step 2 — Review Class Renames.** Classes that likely correspond across versions but have different URIs or names are listed. Each proposal can be confirmed, rejected, or edited. Anything you confirm here is translated into a `DELETE/INSERT` block that rewrites the RDF type of every instance.

**Step 3 — Review Property Renames.** Same logic, applied to attributes, associations, and enum entries, shown in three sub-tabs. This step handles the common case where a property was renamed between CGMES versions without any change to its meaning.

**Step 4 — Review Default Values.** For every property that exists in the *target* schema but not in the *source*, RDFArchitect asks what value to insert when migrating existing instances. Sub-tabs for attributes, associations, and enum entries. You can specify a fixed value, the result of a SPARQL expression, or leave it blank (in which case the target property simply has no value for migrated instances, which may itself be a SHACL violation you need to review).

**Step 5 — Generate Script.** The wizard produces a single `.sparql` file containing all the `DELETE/INSERT WHERE` blocks, in the correct order. A warning is shown that multiplicity changes on associations are not yet handled automatically, and that the migrated data should be validated against the target profile's SHACL afterwards.

The generated script is plain SPARQL and runs on any SPARQL 1.1-compliant endpoint (e.g. Apache Jena Fuseki, the triple store RDFArchitect uses itself).

---

## 12. Sharing and exporting

### Exporting a schema

**File → Export → Schema (RDFS)** exports the currently selected graph to a file. The dialog lets you choose:

- **Format** — RDF/XML (`.rdf`), Turtle (`.ttl`), or N-Triples (`.nt`). RDF/XML is the default for CGMES/ENTSO-E compatibility.
- **Namespaces** — which prefixes to include in the exported file's header.
- **Profile header** — whether to emit the ontology block first (matching the ENTSO-E release convention) and, if so, whether to auto-generate any missing standard entries from the graph metadata.

The exported file is self-contained: it can be re-imported into RDFArchitect, loaded into any SPARQL engine, or handed to downstream CIM tooling.

### Exporting SHACL

See [Exporting SHACL](#exporting-shacl) above. TTL by default.

### Share snapshot

**File → Share Snapshot** creates an immutable snapshot of the currently selected dataset and returns a link of the form `https://<host>/?snapshot=<token>`. Anyone opening that link sees a read-only copy of the dataset at the moment the snapshot was taken — packages, classes, associations, SHACL, everything — and can navigate the schema exactly like the author did, without needing to install anything.

This is the feature to use when you want reviewers to look at a profile without sending RDF files around. Snapshots are stored in the underlying triple store (Fuseki) and persist until the triple store is reset.

Three things to be aware of:

- The snapshot link is *the* access control. Anyone with the link can view.
- Snapshots are read-only by construction — the viewer cannot accidentally edit.
- In the current version, snapshots cannot be deleted via the UI.

---

## 13. Read-only mode

Every dataset has a read-only flag. When the flag is set, all editing actions are disabled in the UI, menu entries switch to their **View** variants, and the save buttons in every dialog are hidden. This is used for two purposes:

- **Protect a dataset** that represents an official, released profile from accidental changes. Use **Edit → Disable Editing** (shown when editing is currently enabled) to lock a dataset.
- **Presentation mode** — the read-only state applied to a snapshot is what makes the shared link safe to send out.

To resume editing, use **Edit → Enable Editing**. Enabling editing does not silently revert anything — it only lifts the write-protection.

---

## 14. Search

The search bar at the top of the editor searches across **classes, attributes, associations, enum entries, and packages**, scoped by default to the currently selected dataset. Results are shown as a ranked list with the matching URI highlighted; clicking a result jumps to that element in the editor (selecting the right dataset, graph, package, and class as needed).

Search is the fastest way to find, for example, every class with `Terminal` in its name across all profiles in a dataset, or every association whose role contains a given fragment.

---

## 15. Tips & keyboard shortcuts

- **Ctrl+Z / Ctrl+Y** — Undo and redo, in every editing context.
- **URL parameters** — The main editor URL accepts `?dataset=...&graph=...&package=...` to jump directly to a given location. This is how deep links from external tools or documents should point at RDFArchitect content.
- **Class context menu** — Right-click on a class in the diagram to focus it (centre + highlight), open its class editor, or hide it.
- **Diagram context menu** — Right-click on empty diagram space for layout actions (auto-layout, fit to view) and to switch between SvelteFlow and Mermaid rendering.
- **Save snapshot before risky changes** — If you are about to try a large migration or a destructive delete, creating a snapshot first gives you a restore point that is independent of the undo history.
- **Filter view** — The view filter dialog can hide external packages or constrain the diagram to a specific stereotype, which keeps large CGMES releases navigable.

---

## Further reading

- [Installation](installation.md) — how to run RDFArchitect locally or via Docker.
- [Features reference](features.md) — a feature-by-feature checklist organised by role.
- [CIM/CGMES concepts mapping](cim-mapping.md) — how RDFArchitect terminology relates to CIM, CGMES, and ENTSO-E concepts.
- [Administrator's guide](administrator-guide.md) — triple store, configuration, backups, read-only datasets.
- [FAQ & troubleshooting](faq.md).

For screenshots of the features described above, see [`docs/screenshots.md`](screenshots.md).
