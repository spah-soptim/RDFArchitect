---
title: CIM/CGMES Mapping
sidebar_position: 2
description: Reference of the RDF predicates RDFArchitect reads and writes, with the canonical CIM/CGMES object structure.
---

# CIM, CGMES, and ENTSO-E Mapping

This page is the reference of the **RDF vocabulary RDFArchitect reads, writes, and round-trips**, plus the canonical structure of CIM-style packages, classes, attributes, associations, and enum entries.

If you've used Sparx EA with the CIM extension or read the CIM/CGMES standards, the predicates below will be familiar. RDFArchitect uses them as the source of truth — every screen in the editor is a UI on top of these triples.

## Predicates in use

### RDFS / RDF

Namespace: `http://www.w3.org/2000/01/rdf-schema#`

| Predicate | Meaning |
| --------- | ------- |
| `rdf:type` | Defines the type of a resource — whether it is a class, an enum entry, or a property (attribute or association). |
| `rdfs:Class` | Marks a resource as a class. |
| `rdfs:subClassOf` | Declares the subject as a sub-class of the object (single inheritance in CIM). |
| `rdfs:label` | Human-readable label for any resource. |
| `rdfs:comment` | Free-text description for any resource. RDFArchitect normalizes the comment's literal datatype to `xsd:string` on import. |
| `rdfs:domain` | Affiliation of a property (attribute / association) to a class. |
| `rdfs:range` | For an enum-typed attribute: the enum class. For an association: the target class. |
| `rdfs:Literal`, `rdfs:Datatype`, `rdf:langString`, `rdf:HTML`, `rdf:XMLLiteral` | Datatype markers that may appear on literal values such as comments. |

### CIMS extension

Namespace: `https://iec.ch/TC57/1999/rdf-schema-extensions-19990926#`

| Predicate | Meaning |
| --------- | ------- |
| `cims:belongsToCategory` | The optional package a class belongs to. |
| `cims:stereotype` | UML-style metadata on a resource — abstract, concrete, primitive, datatype, enumeration, etc. A resource may carry more than one stereotype. |
| `cims:dataType` | The datatype of an attribute, used when the type is **not** an enum class. |
| `cims:multiplicity` | How often an attribute or association can be instantiated. Either a range (`M:0..1`, `M:1..n`) or an exact count (`M:1..1` or `M:1`). Open-ended ranges are allowed (`M:1..n`, `M:n`). |
| `cims:isDefault` | A default value for an attribute. May be a literal or a blank node carrying the literal. |
| `cims:isFixed` | A fixed (constant) value for an attribute. May be a literal or a blank node. |
| `cims:AssociationUsed` | Whether an association should be instantiated in the given direction (`"Yes"` / `"No"`). |
| `cims:inverseRoleName` | IRI of the inverse association. |

## Common stereotype values

`cims:stereotype` is intentionally open-ended; the values RDFArchitect recognises out of the box are:

```
<http://iec.ch/TC57/NonStandard/UML#concrete>
<http://iec.ch/TC57/NonStandard/UML#enumeration>
<http://iec.ch/TC57/NonStandard/UML#attribute>
"Primitive"
"CIMDatatype"
"Entsoe"
```

A class may carry more than one stereotype simultaneously (for example `Primitive` + `CIMDatatype`).

## Ontology header

The standard CGMES 3.0 ontology header predicates are preserved verbatim by RDFArchitect on round-trips and are emitted on the `Ontology` resource at the top of every export:

| Source | Predicates |
| ------ | ---------- |
| Dublin Core (`dct:`) | `dct:conformsTo`, `dct:creator`, `dct:description`, `dct:identifier`, `dct:issued`, `dct:language`, `dct:modified`, `dct:publisher`, `dct:rights`, `dct:rightsHolder`, `dct:title` |
| OWL (`owl:`) | `owl:backwardCompatibleWith`, `owl:incompatibleWith`, `owl:priorVersion`, `owl:versionIRI`, `owl:versionInfo` |
| DCAT (`dcat:`) | `dcat:keyword`, `dcat:landingPage`, `dcat:theme` |

These are not interpreted by the editor — they are surfaced for review and round-tripped on export.

## CIM object structure

The shapes below are what RDFArchitect emits on export and consumes on import. `pre:` is whatever prefix the resource uses; `#required` and `#optional` mark whether the predicate must be present.

### Package

```turtle
pre:Package_{packageName}
    rdf:type        cims:ClassCategory                  ;   # required
    rdfs:label      "{packageName}"@en                  ;   # required
    rdfs:comment    "{comment}"^^{format}               .   # optional
```

### Class

```turtle
pre:{className}
    rdf:type                rdfs:Class                  ;   # required
    rdfs:label              "{className}"@en            ;   # required
    rdfs:subClassOf         {superClassIRI}             ;   # optional (single inheritance)
    rdfs:comment            "{comment}"^^{format}       ;   # optional
    cims:belongsToCategory  {packageIRI}                ;   # optional
    cims:stereotype         {stereotype}                .   # optional, repeatable
```

A class may carry more than one `cims:stereotype` triple. Common values are listed under [Common stereotype values](#common-stereotype-values).

### Attribute

```turtle
pre:{classLabel}.{attributeName}
    rdf:type            rdf:Property                                          ;   # required
    rdfs:label          "{attributeName}"@en                                  ;   # required
    rdfs:domain         {classIRI}                                            ;   # required
    # one of the next two is required:
    rdfs:range          {enumClassIRI}                                        ;   #   for enum-typed attributes
    cims:dataType       {dataTypeIRI}                                         ;   #   for primitive/CIM-datatype attributes
    rdfs:comment        "{comment}"^^{format}                                 ;   # optional
    cims:stereotype     <http://iec.ch/TC57/NonStandard/UML#attribute>        ;   # required
    cims:multiplicity   cims:M:[0-9]+(..(n|[0-9]))                            ;   # required
    cims:isFixed        {value}^^{datatype}
                        | [ rdfs:Literal {value}^^{datatype} ]                ;   # optional
    cims:isDefault      {value}^^{datatype}
                        | [ rdfs:Literal {value}^^{datatype} ]                .   # optional
```

Notes:

- The attribute prefix does **not** have to match the prefix of its domain class.
- Attribute multiplicity is most commonly `M:0..1` (optional) or `M:1..1` (required).
- `cims:isFixed` and `cims:isDefault` may either carry the literal directly or wrap it in a blank node with `rdfs:Literal`. Both forms round-trip.

### Association

An association is a *pair* of `rdf:Property` resources — the forward direction and its inverse — linked by `cims:inverseRoleName`.

Forward direction:

```turtle
pre:{classLabel}.{label}
    rdf:type             rdf:Property                                ;   # required
    rdfs:label           "{label}"@en                                ;   # required (defaults to {targetLabel})
    rdfs:domain          {classIRI}                                  ;   # required
    rdfs:range           {targetIRI}                                 ;   # required
    rdfs:comment         "{comment}"^^{format}                       ;   # optional
    cims:AssociationUsed "Yes" | "No"                                ;   # required
    cims:inverseRoleName {targetIRI}.{inverseLabel}                  ;   # required
    cims:multiplicity    cims:M:[0-9]+(..(n|[0-9]+))?                .   # required
```

Inverse direction:

```turtle
pre:{targetLabel}.{inverseLabel}
    rdf:type             rdf:Property                                ;   # required
    rdfs:label           "{inverseLabel}"@en                         ;   # required (defaults to {classLabel})
    rdfs:domain          {targetIRI}                                 ;   # required
    rdfs:range           {classIRI}                                  ;   # required
    rdfs:comment         "{inverseComment}"^^{format}                ;   # optional
    cims:AssociationUsed "Yes" | "No"                                ;   # required
    cims:inverseRoleName {classIRI}.{label}                          ;   # required
    cims:multiplicity    cims:M:[0-9]+(..(n|[0-9]+))?                .   # required
```

Notes:

- The forward and inverse association may sit in different namespaces from each other and from the domain/range classes.
- `cims:AssociationUsed = "No"` records the direction in the schema but suppresses it in instantiation.

### Enum entry

```turtle
pre:{classLabel}.{enumEntryName}
    rdf:type        {enumClassIRI}                  ;   # required
    rdfs:label      "{enumEntryName}"@en            ;   # required
    rdfs:comment    "{comment}"^^{format}           ;   # optional
    cims:stereotype "enum"                          .   # optional
```

The enum entry's `rdf:type` points to the enum **class** (not `rdfs:Class`), which is how enum membership is detected on import.

## How RDFArchitect concepts map to CIM/CGMES

| RDFArchitect | CIM / CGMES / ENTSO-E equivalent | Notes |
| ------------ | -------------------------------- | ----- |
| **Dataset** | n/a (deployment-level) | Equivalent to a Fuseki dataset; not a CIM concept. |
| **Graph** | A CIM **profile** (e.g. `EquipmentProfile`) | One-to-one for typical CGMES use. |
| **Package** | UML **Package** (`cims:ClassCategory`) | Identical concept and serialization. |
| **Class** | UML **Class** (`rdfs:Class`) | Identical. |
| **Attribute** | UML **Attribute** (`rdf:Property` with literal range) | Identical. |
| **Association** | UML **Association** (`rdf:Property` with class range) | Stored as a forward + inverse pair joined by `cims:inverseRoleName`. |
| **Enumeration** | UML **Enumeration** | Class with stereotype `enumeration`; entries typed by the enum class itself. |
| **Enum entry** | UML **EnumLiteral** | First-class IRI. |
| **Stereotype** | UML stereotype | `cims:stereotype` predicate. |
| **Multiplicity** | UML cardinality | `cims:multiplicity` literal. |
| **SHACL constraint** | OCL constraint (CIM) / SHACL profile (CGMES 3.0) | RDFArchitect uses SHACL natively; OCL is round-tripped but not interpreted. |

## CGMES profiles

CGMES is organised into profiles (Equipment, Topology, State Variables, Steady State Hypothesis, Geographical Location, Diagram Layout, Dynamics).

In RDFArchitect, each profile is one **graph** in a dataset. The dataset acts as the CIM extension or company portfolio. Common patterns:

- One dataset for the upstream CIM/CGMES baseline (read-only after import).
- One dataset per project, with the in-house extension on top of the baseline.
- Snapshots taken at each CGMES release boundary for auditability.

## ENTSO-E network code profiles

ENTSO-E publishes additional profiles on top of CGMES (Reporting, Operational Planning, Capacity Calculation Region, …). These are imported the same way — one Turtle file per profile, one graph per profile. RDFArchitect doesn't bake in any ENTSO-E-specific assumptions; it treats every imported file the same way.

## Migration path between CIM versions

Use the [Migration Wizard](/user-guide/migration-wizard). The supplied SPARQL templates handle the standard kinds of CIM-version delta: class renames, attribute renames and datatype changes, association renames, enum entry additions/removals/renames, domain (target class) changes, and property deletions.

For an ENTSO-E profile that adds a new mandatory property, the wizard's step 4 ("Confirm defaults for new mandatory properties") is where you provide values.

## What RDFArchitect does *not* aim to be

- A CIM **instance** editor (no instance data CRUD UI). Use the migration wizard's output against your own triple store.
- An OCL evaluator. SHACL is the constraint language; OCL constraints are imported as opaque triples and round-tripped, but not interpreted.
- A profile *builder* in the UML-modelling-tool sense. RDFArchitect edits at the RDFS level — appropriate for the post-CIM-Sparx-EA workflow, not for upstream CIM core development at IEC.
