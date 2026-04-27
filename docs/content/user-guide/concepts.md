---
title: Core Concepts
sidebar_position: 2
---

# Core Concepts

RDFArchitect is built around a small set of concepts borrowed from the RDF and UML worlds. Once these click, every screen in the tool reads naturally.

## Dataset

A **dataset** is a top-level container, equivalent to a *named dataset* in an RDF triple store. In a default installation it is backed by an Apache Jena Fuseki dataset.

Use one dataset per project, customer, or compliance scope. Datasets are the unit of access control, backup, and bulk import/export. The default dataset is named `default`.

## Graph

A **graph** is a *named graph* inside a dataset and corresponds to a single schema (or schema profile). Most editing happens at the graph level:

- Importing a Turtle/RDF/XML file creates or replaces a graph.
- Comparing two schemas compares two graphs.
- The changelog, snapshots, and version history are per-graph.
- The "active graph" indicator in the toolbar is what every editor action targets.

A dataset can hold many graphs side-by-side — for example one graph per CIM/CGMES profile.

## Package

A **package** groups classes that belong together logically. Packages are the primary navigation aid: the left-hand tree groups classes by package, and the diagram view focuses one package at a time.

Packages are stored in the graph as `cims:Package` resources and behave like UML packages. A class always belongs to exactly one package.

## Class

A **class** is the central modeling element. In CIM terms, every `Breaker`, `BaseVoltage`, `Terminal`, etc. is a class. In RDFArchitect a class carries:

- An IRI and a human-readable label.
- Zero or one parent class (single inheritance).
- A set of **attributes** (typed literal properties).
- A set of **associations** (object properties pointing to other classes).
- For enumerations: a set of enum entries.
- A package assignment.
- Optional descriptive comment.
- Optional class-level SHACL constraints.

## Attribute

An **attribute** is a literal-valued property: a name, a datatype (string, integer, boolean, date, …), a multiplicity (cardinality), and optional metadata such as a fixed value or stereotype. Attributes are the leaves of the model.

## Association

An **association** points from one class to another. It carries a name, a multiplicity on each end, and (optionally) a direction and a stereotype such as `aggregate` or `composite`. Associations show up as edges in the diagram.

## Enumeration

An **enumeration** is a class whose only purpose is to define a fixed set of allowed values (e.g. `BreakerType` with entries `Air`, `Vacuum`, `SF6`). Enum entries are first-class citizens in RDFArchitect: they have IRIs and can be referenced from attributes.

## SHACL Shapes

[**SHACL**](https://www.w3.org/TR/shacl/) (Shapes Constraint Language) is the W3C standard for validating RDF data. RDFArchitect treats SHACL in two complementary ways:

- **Generated shapes** are derived from your model automatically. Every class becomes a `sh:NodeShape`, every attribute and association becomes a `sh:PropertyShape`, with cardinalities and datatypes filled in.
- **Custom shapes** are SHACL files you author yourself (or import from another tool). They live alongside the generated shapes and can express constraints that don't fit naturally into RDFS — pattern matches, value ranges, sh:sparql constraints, and so on.

Both kinds are inspectable from the SHACL views and from the class editor.

## Namespace and Prefix

A **namespace** is the IRI base used for resources in your schema (e.g. `http://iec.ch/TC57/CIM100#`). A **prefix** is the short name you use for it in serialized RDF (`cim:`).

RDFArchitect manages prefixes both **globally** (defaults applied to every new graph) and **per-dataset** (overrides for a specific dataset). See [Prefixes](./prefixes).

## Snapshot

A **snapshot** is a frozen, read-only copy of a graph at a point in time. Snapshots are designed for sharing: you can hand a snapshot link to a reviewer who has no editing access, and they get a fully browsable view of the schema, packages, classes, and SHACL constraints.

## Changelog

Every write operation against a graph appends an entry to that graph's **changelog**. From the Changelog view you can:

- See who changed what and when.
- Inspect the before/after of a change.
- Restore the graph to an earlier point in time.
- Undo or redo recent operations from the toolbar.

## Read-only Mode

A graph (or an entire dataset) can be marked **read-only**. Read-only graphs are perfectly browsable — diagrams, classes, SHACL, comparison — but every editing affordance is hidden or disabled. Snapshots are always read-only.

## How they fit together

```text
Dataset ─┬─ Graph ─┬─ Package ─── Class ─┬─ Attribute
         │         │                     ├─ Association
         │         │                     └─ Enum entry (for enumerations)
         │         ├─ Custom SHACL shapes
         │         ├─ Changelog
         │         └─ Snapshots
         └─ Namespaces / Prefixes
```

Most things you do in RDFArchitect — importing, editing, validating, comparing, sharing — are operations on a single graph inside a dataset.
