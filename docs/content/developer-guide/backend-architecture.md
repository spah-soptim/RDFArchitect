---
title: Backend Architecture
sidebar_position: 5
---

# Backend Architecture

The backend is a Spring Boot 4 application running on Java 25, organised as a hexagonal (ports-and-adapters) architecture.

## Layered overview

```
HTTP Request
   │
   ▼
┌─────────────────────────────────────────┐
│ api/controller/                         │  Thin REST controllers
│   └─ delegate to use cases              │
├─────────────────────────────────────────┤
│ services/<feature>/                     │  Use case interfaces + impls
│   ├─ business logic                     │
│   └─ orchestrates side effects          │
├─────────────────────────────────────────┤
│ database/ + graph/                      │  Ports & adapters
│   ├─ DatabasePort, GraphPort            │
│   ├─ Fuseki, file, in-memory adapters   │
│   └─ Jena transaction wrappers          │
└─────────────────────────────────────────┘
   │
   ▼
Apache Jena (RDF dataset, SPARQL, SHACL)
```

A request lands on a controller, the controller calls a use-case interface, the implementation orchestrates one or more port operations, and the ports talk to the actual datastore. Tests can exercise the same use-case implementations against an in-memory adapter without spinning up Fuseki.

## Controllers (`api/controller/`)

REST controllers are deliberately thin. Their responsibilities are limited to:

- Mapping HTTP verbs and paths to use-case calls.
- Decoding/encoding DTOs via MapStruct mappers.
- Surfacing exceptions as appropriate HTTP status codes via the global `@ControllerAdvice`.
- Audit logging for write operations.

A controller skeleton looks like this:

```java
@RestController
@RequestMapping("/api/datasets/{datasetId}/graphs/{graphId}/classes")
@RequiredArgsConstructor
public class ClassRESTController {

    private final ListClassesUseCase listClasses;
    private final EditClassUseCase editClass;
    private final ClassMapper mapper;

    @GetMapping
    public Response<List<ClassDTO>> list(@PathVariable String datasetId,
                                         @PathVariable String graphId) {
        var graph = new GraphIdentifier(datasetId, graphId);
        var classes = listClasses.execute(graph);
        return Response.ok(mapper.toDtoList(classes));
    }

    @PutMapping("/{classIri}")
    public Response<ClassDTO> edit(@PathVariable String datasetId,
                                   @PathVariable String graphId,
                                   @PathVariable String classIri,
                                   @RequestBody EditClassDTO request) {
        var graph = new GraphIdentifier(datasetId, graphId);
        var updated = editClass.execute(graph, classIri, mapper.fromDto(request));
        return Response.ok(mapper.toDto(updated));
    }
}
```

Notes:

- Controllers receive raw IRI strings and delegate IRI expansion to the existing helpers — never roll your own URL decoder.
- All write methods feed the changelog via the use-case implementation, not at controller level.

## Use cases (`services/<feature>/`)

For each feature there is a `*UseCase` interface and at least one implementation. The interface is the contract used by controllers and tests; the implementation contains the orchestration:

```java
public interface EditClassUseCase {
    Class execute(GraphIdentifier graph, String classIri, ClassEdit edit);
}

@Service
@RequiredArgsConstructor
public class EditClassService implements EditClassUseCase {

    private final GraphPort graphPort;
    private final ChangelogService changelog;
    private final LayoutService layout;
    private final ReadOnlyService readOnly;

    @Override
    public Class execute(GraphIdentifier graph, String classIri, ClassEdit edit) {
        readOnly.assertWritable(graph);
        try (var txn = graphPort.beginWrite(graph)) {
            var before = txn.findClass(classIri);
            var after = txn.applyClassEdit(classIri, edit);
            changelog.record(txn, classIri, before, after);
            layout.touch(txn, classIri);
            txn.commit();
            return after;
        }
    }
}
```

Why interfaces? Because there are roughly 80 of them across the backend; they look near-identical, but the indirection lets us:

- Unit-test controllers by mocking the interface.
- Swap in alternative implementations (e.g. an audited variant) without touching callers.
- Keep dependency direction one-way: controller → interface → implementation.

When you add a new feature, follow the same pattern even if the implementation is trivial.

## DTOs and mappers (`api/dto/`)

Every HTTP request and response shape is modelled as a DTO record (or POJO). MapStruct-generated mappers convert between DTOs and the internal domain types. Hand-written conversion is acceptable only when the structural distance is too large for MapStruct.

DTOs live next to their controllers in the package hierarchy. The MapStruct mapper interface annotated with `@Mapper(componentModel = "spring")` is picked up automatically.

## Database layer (`database/`, `graph/`)

`DatabasePort` is the top-level abstraction over the dataset. `GraphPort` operates on a single graph. There are three adapters:

| Adapter | Use |
| ------- | --- |
| `FusekiDatabase` | Production. Talks to a remote Fuseki via HTTP. |
| `FileDatabase` | Single-process file-backed mode. |
| `InMemoryDatabase` | Unit and integration tests. |

The `databaseType` property in `application-database.yml` selects which adapter is wired in.

### Transactions

All graph mutations follow the same pattern:

```java
try (var txn = graphPort.beginWrite(graph)) {
    // perform reads and writes via txn
    txn.commit();
}
```

`try-with-resources` guarantees `end()` is called even if `commit()` is missed. Every Jena `Dataset` operation lives inside a transaction — *do not* call low-level Jena APIs directly from a service; route them through the port.

### `GraphIdentifier`

Every operation that targets a single graph takes a `GraphIdentifier(datasetId, graphId)`. This type is the canonical representation throughout the backend; never pass two raw strings around.

## SHACL pipeline (`shacl/`)

The SHACL package contains:

- The **generator** that walks the model and emits `sh:NodeShape` / `sh:PropertyShape` triples.
- The **importer** that classifies incoming SHACL into custom shapes for storage.
- The **exporter** that combines generated and stored shapes for output.

The generator is deterministic — given the same model, it produces byte-for-byte identical output. This is the property that makes it safe to *not* persist generated shapes.

## Migration template composer (`migration/`)

The migration wizard's last step is implemented here. The composer:

1. Reads the comparison result.
2. Loads SPARQL templates from `resources/sparql-templates/migration/`.
3. Substitutes class/property IRIs into the templates.
4. Concatenates the blocks into a single SPARQL Update file.

Each template handles one kind of change (class rename, property delete, attribute datatype change, …). Adding a new pattern means adding a new template and a corresponding entry in the composer.

## Exception handling

A single `@ControllerAdvice` translates domain exceptions to HTTP status codes:

- `NotFoundException` → 404
- `ReadOnlyException` → 409
- `ValidationException` → 400
- everything else → 500 (logged with stack trace)

Adding a new exception means adding a new `@ExceptionHandler` method or extending an existing one.

## Audit logging

Every write controller logs a structured audit line: timestamp, principal (if any), graph identifier, operation, target. The changelog feature reads these entries — the audit log is **the source of truth** for history, not a redundant copy.
