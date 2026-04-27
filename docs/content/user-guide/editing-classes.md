---
title: Editing Classes
sidebar_position: 6
---

# Editing Classes

The class editor is the primary editing surface in RDFArchitect. This page is a complete reference for what you can do inside it.

![Class editor](/img/screenshots/class-editor.png)

## Opening the editor

Three ways to open a class:

- Double-click a class in the diagram.
- Right-click a class in the diagram and choose **Edit class**.
- Click a class name in the package navigation tree.

The editor opens in a side panel and stays open as you click around — switching to another class swaps the contents in place.

## Layout

The editor is organised into tabs:

| Tab | What it shows |
| --- | ------------- |
| **General** | Label, IRI, parent class, package, comment, abstract flag, stereotype. |
| **Attributes** | List of literal-valued properties on this class. |
| **Associations** | List of object properties pointing from this class. |
| **Enum entries** | (Only for enumerations) The set of allowed values. |
| **SHACL** | Generated and custom SHACL shapes that target this class. |

The **General** tab is loaded first; the others are populated lazily.

## General tab

| Field | Notes |
| ----- | ----- |
| **Label** | Free-text human-readable name. Stored as `rdfs:label`. |
| **IRI** | The class identifier. Editable but must remain unique inside the graph. |
| **Package** | Drop-down of all packages in the graph. Required. |
| **Parent class** | Single-inheritance pointer (`rdfs:subClassOf`). May be empty. |
| **Stereotype** | Free-text or controlled value (`«enumeration»`, `«primitive»`, `«CIMDatatype»`, …). |
| **Abstract** | Marks the class as abstract; affects validation and SHACL targeting. |
| **Comment** | Multi-line description; stored as `rdfs:comment`. |

Renaming a class updates every association that refers to it (in the same graph). The change is recorded in the changelog with both old and new IRI.

## Attributes tab

Each row is one attribute and supports inline editing of:

- **Name** (becomes the IRI fragment).
- **Datatype** — a drop-down sourced from XSD plus the CIM primitives. Pick `xsd:string`, `xsd:integer`, `xsd:dateTime`, `cim:Float`, `cim:Boolean`, `cim:PerCent`, etc.
- **Multiplicity** — `0..1`, `1..1`, `0..*`, `1..*`, or a custom range.
- **Fixed value** — optional default/constant.
- **Stereotype** — free text.
- **Comment** — per-attribute description.

Adding an attribute creates a new `rdf:Property` resource with the appropriate domain. Removing one preserves the deletion in the changelog.

## Associations tab

Associations are object properties. Each one carries:

- **Name**.
- **Target class** — chosen from a searchable drop-down of classes in the graph.
- **Source multiplicity** and **target multiplicity** (e.g. `1..1` ↔ `0..*`).
- **Stereotype** — `aggregate`, `composite`, or empty.
- **Comment**.

Associations show up as labelled edges in the diagram, with arrowheads reflecting the navigability.

## Enum entries tab

Visible only when the class is an enumeration. Each entry has:

- **Name** (becomes the IRI fragment).
- **Comment**.

Order is preserved so an enumeration can be displayed and exported in a stable sequence.

## SHACL tab

Lists every shape that targets the current class:

- **Generated shapes** are auto-derived from the schema (one node shape, plus property shapes per attribute and association). They are recomputed every time the model changes.
- **Custom shapes** that target this class are listed alongside, clearly distinguished.

Clicking a property shape reveals its constraints (datatype, min/max count, sh:in, sh:pattern, etc.). See [SHACL](./shacl) for the full picture.

## Saving and validation

Each tab has its own save action. RDFArchitect runs in-tab validity checks before allowing save:

- Empty mandatory fields are highlighted.
- Duplicate IRIs (across the whole graph) are blocked.
- Multiplicity strings are parsed; malformed values are rejected.

Unsaved changes are kept locally as long as the editor is open. Switching to another class will warn you about pending edits before discarding them.

## Deleting a class

The delete action is in the editor's overflow menu. A confirmation dialog asks you to acknowledge that:

- All attributes and associations on this class will be deleted.
- All associations from *other* classes that target this one will be deleted.
- The deletion is reversible via the changelog (until older history is pruned).

## Keyboard shortcuts

| Shortcut | Action |
| -------- | ------ |
| `Ctrl/Cmd + S` | Save the current tab. |
| `Esc` | Close the editor (warns about unsaved changes). |
| `Ctrl/Cmd + Z` | Undo last operation (works globally on the active graph). |
| `Ctrl/Cmd + Shift + Z` | Redo. |
| `/` | Focus the global search bar. |
