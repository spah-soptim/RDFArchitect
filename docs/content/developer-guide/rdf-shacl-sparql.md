---
title: RDF, SHACL, and SPARQL
sidebar_position: 9
---

# Working With RDF, SHACL, and SPARQL

Where in the codebase to land RDF-, SHACL-, or SPARQL-shaped work.

## Apache Jena entry points

The backend uses [Apache Jena 5](https://jena.apache.org/). The primary types you'll touch are:

| Type | Purpose |
| ---- | ------- |
| `Dataset` | The Fuseki dataset wrapper. Held inside `DatabasePort`. |
| `Model` | A single named graph. Obtained via `dataset.getNamedModel(graphIri)`. |
| `Resource`, `Property`, `Literal` | Triple components. |
| `RDFNode` | Common parent type. |
| `QueryExecution` / `UpdateExecution` | SPARQL execution. |

Always go through the project's `GraphPort` instead of hitting Jena directly. The port handles transactions, snapshot routing, and read-only enforcement consistently.

## Reading data

```java
try (var txn = graphPort.beginRead(graph)) {
    var query = sparql.load("get-subclasses");
    var results = txn.executeSelect(query, row -> row.get("subclass").asResource().getURI());
}
```

`executeSelect` returns a `List<T>`. For ASK or CONSTRUCT, see `executeAsk` / `executeConstruct` in the same port.

## Writing data

```java
try (var txn = graphPort.beginWrite(graph)) {
    var update = sparql.load("rename-class").bind("oldIri", oldIri).bind("newIri", newIri);
    txn.executeUpdate(update);
    changelog.record(txn, oldIri, /* before */ null, /* after */ null);
    txn.commit();
}
```

Two non-negotiable rules:

1. Every mutation goes through a write transaction.
2. Every mutation that touches user-visible model state records a changelog entry.

## SPARQL templates

Templates live under `backend/src/main/resources/sparql-templates/`. The `SparqlTemplate` helper:

- Loads templates by name from the classpath.
- Substitutes parameters via `?varName` placeholders.
- Caches parsed queries.

Templates are plain `.sparql` files. They're the right place for *every* non-trivial query in the application, for two reasons:

- They're auditable — operators can see what queries the application runs.
- They're testable — they can be exercised against an in-memory dataset directly.

## SHACL generator

`backend/src/main/java/org/rdfarchitect/shacl/` contains the generator. The pipeline is:

1. `SchemaCollector` walks the graph, producing a flat list of classes, attributes, associations, and enums.
2. `NodeShapeBuilder` emits one `sh:NodeShape` per class.
3. `PropertyShapeBuilder` emits property shapes for each attribute and association.
4. `EnumShapeBuilder` emits `sh:in` constraints for enumerations.
5. The shape model is merged with custom shapes loaded from the graph.

Generated shapes are deterministic — given the same model, the output is byte-identical. This is why we don't persist them.

## Adding a SHACL constraint kind

If you need the generator to emit a new constraint:

1. Decide whether it's structural (extend `PropertyShapeBuilder`) or class-level (extend `NodeShapeBuilder`).
2. Add the new shape predicate to the model emission.
3. Update the SHACL importer to recognise the same predicate when importing files.
4. Add a fixture-based test under `shacl/` that covers a model exhibiting the constraint.

Don't fork a new builder — extend the existing ones so generated and imported shapes stay symmetric.

## Migration template composer

`backend/src/main/java/org/rdfarchitect/migration/` reads the comparison result and emits a SPARQL Update script. Templates per migration kind live in `resources/sparql-templates/migration/`:

```
migration/
├── class-renamed.sparql
├── class-deleted.sparql
├── attribute-added.sparql
├── attribute-renamed.sparql
├── attribute-datatype-changed.sparql
├── attribute-fixed-value-changed.sparql
├── association-added.sparql
├── association-renamed.sparql
├── domain-renamed.sparql
├── enum-entry-renamed.sparql
├── enum-entry-deleted.sparql
└── property-deleted.sparql
```

Each template handles one kind of detected difference. Adding a new pattern means:

1. Adding a new template file.
2. Adding the corresponding `MigrationStep` enum value.
3. Wiring it up in the composer.
4. Surfacing it in the wizard's UI (one of the five steps).

## Diagram layout vocabulary

The diagram engine writes layout triples in the `arch:` namespace. See [RDF Data Model](./data-model). Layout updates use the same write-transaction-plus-changelog pattern as model edits, so resetting a layout is undoable.

## Things to be careful about

- **Blank nodes.** Jena assigns fresh blank-node IDs on import. Round-trip identity for blank nodes is by *structure*, not by identifier. The exporter sorts blank nodes deterministically so diffs stay small.
- **Datatype precision.** `xsd:integer` and `xsd:int` are not the same type to Jena. Don't normalise them silently — the schema author may have intended a specific one.
- **Order.** RDF is a set, not a sequence. Where the user perceives order (enum entries, attribute display order), it's stored explicitly via `arch:` ordinal predicates.
- **Whitespace in literals.** Comments contain newlines and indentation that should round-trip. The compare engine normalises whitespace for diffing but never normalises it on disk.
