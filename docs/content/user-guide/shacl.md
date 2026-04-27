---
title: SHACL
sidebar_position: 8
---

# SHACL Constraints

SHACL ([Shapes Constraint Language](https://www.w3.org/TR/shacl/)) is the W3C standard for validating RDF data. RDFArchitect treats SHACL as a first-class part of the schema and supports two kinds of shapes side by side: **generated** and **custom**.

![SHACL view](/img/screenshots/shacl.png)

## Generated vs. custom shapes

| | Generated | Custom |
| - | --------- | ------ |
| **Source** | Auto-derived from the schema. | Authored by you (or imported). |
| **Lifecycle** | Recomputed every time the model changes. | Persists in the graph until you change it. |
| **Editable in UI** | No — change the schema instead. | Yes — via SHACL upload or external editing. |
| **Persisted** | Not stored in the graph. | Stored in the graph alongside the schema. |
| **Exported** | Yes (when you select "include generated"). | Yes, always. |

## What gets generated

For every class in the model RDFArchitect emits:

- A `sh:NodeShape` with `sh:targetClass` pointing to the class.
- One `sh:property` per attribute, including `sh:datatype`, `sh:minCount`, and `sh:maxCount` derived from the multiplicity.
- One `sh:property` per association, with `sh:class` and the same cardinality treatment.
- For enumerations, a `sh:in` constraint listing the allowed entries.
- Cardinality constraints inherited from the parent class.

This means a typical CIM class produces ~10–30 generated shapes — enough to validate that instance data matches the structural model without any manual SHACL authoring.

## What custom shapes are for

Generated shapes cover *structural* constraints. Custom shapes are where you express constraints that are not derivable from the schema:

- Pattern matches (`sh:pattern` for ID formats).
- Value ranges (`sh:minInclusive`, `sh:maxInclusive`).
- Cross-property invariants via `sh:sparql`.
- Conditional shapes via `sh:and`, `sh:or`, `sh:xone`, `sh:not`.
- Domain-specific severity levels and messages.
- Anything else you'd reach for in a hand-written SHACL file.

## Inspecting SHACL

Three views, depending on the question you have:

### Class-specific view

Inside the class editor, the **SHACL** tab shows everything that targets the current class. Generated shapes are tagged ★ generated, custom shapes have a ✎ marker. Property shapes are grouped by the property they constrain.

This is the right view when reviewing a single class.

### Property-specific view

When you click a property inside the class SHACL tab, a smaller dialog focuses on that one property: every shape (across the schema) that constrains it, with its severity, message, and any custom predicates.

This is the right view when investigating "why is this attribute being flagged".

### Full graph view

From the SHACL menu, **Full SHACL view** shows every shape in the graph in one searchable list. Filter by:

- Generated / custom.
- Target class.
- Property path.
- Constraint type.

This is the right view when auditing the constraint surface as a whole.

## Importing SHACL

You can either:

- **Import a file with SHACL inside the schema.** Any `sh:NodeShape` / `sh:PropertyShape` reachable from `rdf:type` is recognised and stored as custom shapes.
- **Import SHACL into an existing graph** through the dedicated SHACL upload dialog, which adds the shapes without touching the existing schema content.

Conflicts (e.g. an imported shape with the same IRI as an existing one) are surfaced in the upload dialog before they are committed.

## Exporting SHACL

The SHACL export dialog lets you choose:

- **Generated**, **custom**, or **both**.
- The serialization (Turtle is the default).
- Whether to bundle the schema alongside.

Exporting "both" produces a file that any standard SHACL validator (Apache Jena's `shacl validate`, TopBraid SHACL, pySHACL, …) will accept directly.

## Severity and messages

Custom shapes can carry `sh:severity` (`sh:Violation`, `sh:Warning`, `sh:Info`) and `sh:message` per language tag. Both are surfaced in the inspection views and preserved on round-trip.

Generated shapes are always emitted with `sh:Violation` severity and a templated message identifying the constraint origin (e.g. *"Missing required attribute X on instance of Y"*).

## Working without SHACL

If you don't care about constraints, you can ignore the SHACL tabs entirely. Generated shapes are produced on demand and never bloat your stored graph; custom shapes only exist if you import them.

## Limitations in 1.0

- The UI does not yet let you author custom SHACL inline — you author shapes in a Turtle file and import them.
- Validating instance data against the shapes happens outside RDFArchitect (any standard SHACL engine works). A built-in instance-validator is on the roadmap.
- `sh:sparql` constraints are stored and exported faithfully but are not previewed in the inspection UI beyond their raw SPARQL string.
