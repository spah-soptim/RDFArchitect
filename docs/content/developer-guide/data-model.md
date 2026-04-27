---
title: RDF Data Model
sidebar_position: 7
---

# RDF Data Model

A reference for the vocabulary RDFArchitect reads, writes, and round-trips. If you're working on the backend graph layer, the SHACL generator, or the migration composer, this is your spec.

## Namespaces in use

| Prefix | IRI | Used for |
| ------ | --- | -------- |
| `rdf` | `http://www.w3.org/1999/02/22-rdf-syntax-ns#` | Standard RDF terms. |
| `rdfs` | `http://www.w3.org/2000/01/rdf-schema#` | Class/property metadata. |
| `owl` | `http://www.w3.org/2002/07/owl#` | Used for `owl:Class`, `owl:Ontology`. |
| `xsd` | `http://www.w3.org/2001/XMLSchema#` | Datatypes. |
| `sh` | `http://www.w3.org/ns/shacl#` | SHACL. |
| `cim` | `http://iec.ch/TC57/CIM100#` (project-configurable) | CIM model. |
| `cims` | `http://iec.ch/TC57/1999/rdf-schema-extensions-19990926#` | CIM extension annotations. |
| `arch` | `https://rdfarchitect.de/ns/architect#` | Internal annotations (diagram layout, snapshots). |

## Class

A class is an `rdfs:Class` (or `owl:Class`):

```turtle
cim:Breaker
    a               rdfs:Class ;
    rdfs:label      "Breaker" ;
    rdfs:comment    "A mechanical switching device." ;
    rdfs:subClassOf cim:Switch ;
    cims:belongsToCategory cim:Wires ;
    cims:stereotype "concrete" .
```

| Predicate | Meaning |
| --------- | ------- |
| `rdfs:label` | Display name. |
| `rdfs:comment` | Free-text description. |
| `rdfs:subClassOf` | Single-inheritance pointer. |
| `cims:belongsToCategory` | Package membership. |
| `cims:stereotype` | UML stereotype (`abstract`, `enumeration`, `primitive`, `CIMDatatype`, …). |

## Attribute

An attribute is an `rdf:Property` whose range is a literal datatype:

```turtle
cim:Breaker.ratedCurrent
    a              rdf:Property ;
    rdfs:label     "ratedCurrent" ;
    rdfs:domain    cim:Breaker ;
    cims:dataType  cim:CurrentFlow ;
    cims:multiplicity "M:0..1" ;
    cims:stereotype "attribute" .
```

`cims:multiplicity` values follow the CIM convention: `M:0..1`, `M:1..1`, `M:0..n`, `M:1..n`, or a literal range like `M:2..5`.

## Association

An association is an `rdf:Property` whose range is another class:

```turtle
cim:Breaker.Terminals
    a              rdf:Property ;
    rdfs:label     "Terminals" ;
    rdfs:domain    cim:Breaker ;
    rdfs:range     cim:Terminal ;
    cims:multiplicity "M:1..n" ;
    cims:inverseRoleName "Breaker" ;
    cims:stereotype "byreference" .
```

`cims:inverseRoleName` records the inverse association name when one exists.

## Enumeration

An enumeration is a class with stereotype `enumeration`. Its entries are individuals:

```turtle
cim:BreakerType
    a               rdfs:Class ;
    rdfs:label      "BreakerType" ;
    cims:stereotype "enumeration" .

cim:BreakerType.Air        a cim:BreakerType ; rdfs:label "Air" .
cim:BreakerType.Vacuum     a cim:BreakerType ; rdfs:label "Vacuum" .
cim:BreakerType.SF6        a cim:BreakerType ; rdfs:label "SF6" .
```

## Package

A package is a `cims:ClassCategory`:

```turtle
cim:Wires
    a            cims:ClassCategory ;
    rdfs:label   "Wires" ;
    cims:belongsToCategory cim:Domain .
```

Packages can nest via `cims:belongsToCategory`.

## Diagram layout

Layout is annotation-only and lives in the `arch:` namespace:

```turtle
cim:Breaker
    arch:layoutX 240 ;
    arch:layoutY 130 ;
    arch:width   180 ;
    arch:height  90 .
```

These triples are produced and consumed only by RDFArchitect itself.

## Generated SHACL

For a class with one attribute, the generator emits:

```turtle
cim:BreakerShape
    a              sh:NodeShape ;
    sh:targetClass cim:Breaker ;
    sh:property [
        sh:path     cim:Breaker.ratedCurrent ;
        sh:datatype cim:CurrentFlow ;
        sh:minCount 0 ;
        sh:maxCount 1 ;
    ] .
```

Generated shapes carry an internal marker (`arch:generated true`) so the importer can distinguish them from custom shapes on round-trip; the marker is stripped on export.

## Custom SHACL

Anything else under `sh:NodeShape` / `sh:PropertyShape` is treated as custom. Custom shapes are stored verbatim in the graph and round-trip unchanged.

## Snapshot graphs

Snapshots are stored as named graphs whose IRI is built from the source graph plus a snapshot ID:

```
arch:snapshots/{datasetId}/{graphId}/{snapshotId}
```

The snapshot graph is a copy of the source graph at creation time, plus an `arch:snapshotMetadata` resource holding name, description, author, and timestamp.

## Changelog graphs

Each graph has a sibling changelog graph at `arch:changelog/{datasetId}/{graphId}` storing one resource per recorded change. Each change carries:

- `arch:targetIri` — the resource that changed.
- `arch:operation` — `create`, `update`, `delete`.
- `arch:beforeTriples` / `arch:afterTriples` — Turtle blobs (literal-encoded) capturing the diff.
- `arch:timestamp` — change time.
- `arch:author` — principal, when available.

## Conventions for new vocabulary

If you need to add a new annotation:

- Use the `arch:` namespace for *internal* annotations that should not surface to interoperating tools.
- Use the `cims:` namespace only when adding annotations that exist in the CIM specs.
- Document the new term here and in the SHACL generator if it affects validation.
