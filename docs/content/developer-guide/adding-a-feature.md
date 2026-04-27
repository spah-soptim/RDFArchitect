---
title: Adding a Feature End-to-End
sidebar_position: 8
---

# Adding a Feature End-to-End

A worked example: **"List all abstract classes in a graph"**. Eight layers, all eight touched. Use this as a template when you add your own feature.

## The shape of the change

| Layer | File | What you add |
| ----- | ---- | ------------ |
| 1. Use case interface | `services/classes/ListAbstractClassesUseCase.java` | Declares the operation. |
| 2. Use case implementation | `services/classes/ListAbstractClassesService.java` | The actual SPARQL/Jena code. |
| 3. SPARQL template | `resources/sparql-templates/list-abstract-classes.sparql` | The query. |
| 4. DTO | `api/dto/classes/AbstractClassDTO.java` | Response shape. |
| 5. Mapper | `api/dto/classes/AbstractClassMapper.java` | Domain → DTO. |
| 6. Controller | `api/controller/datasets/graphs/classes/AbstractClassRESTController.java` | Endpoint. |
| 7. Frontend API method | `frontend/src/lib/api/backend.js` | Call wrapper. |
| 8. Frontend usage | `routes/mainpage/...` | UI. |

Plus tests at every layer that has logic.

## 1. Use case interface

```java
// backend/src/main/java/org/rdfarchitect/services/classes/ListAbstractClassesUseCase.java
package org.rdfarchitect.services.classes;

import java.util.List;
import org.rdfarchitect.graph.GraphIdentifier;
import org.rdfarchitect.model.Class;

public interface ListAbstractClassesUseCase {
    List<Class> execute(GraphIdentifier graph);
}
```

## 2. Use case implementation

```java
@Service
@RequiredArgsConstructor
public class ListAbstractClassesService implements ListAbstractClassesUseCase {

    private final GraphPort graphPort;
    private final SparqlTemplate sparql;

    @Override
    public List<Class> execute(GraphIdentifier graph) {
        try (var txn = graphPort.beginRead(graph)) {
            var query = sparql.load("list-abstract-classes");
            return txn.executeSelect(query, row -> Class.builder()
                    .iri(row.get("iri").asResource().getURI())
                    .label(row.get("label").asLiteral().getString())
                    .build());
        }
    }
}
```

Read transactions are fine for read-only operations.

## 3. SPARQL template

```sparql
# resources/sparql-templates/list-abstract-classes.sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX cims: <http://iec.ch/TC57/1999/rdf-schema-extensions-19990926#>

SELECT ?iri ?label
WHERE {
  ?iri  a              rdfs:Class ;
        rdfs:label     ?label ;
        cims:stereotype "abstract" .
}
ORDER BY ?label
```

Keep templates side-by-side under `sparql-templates/`. The `SparqlTemplate` helper handles classpath loading.

## 4. DTO

```java
public record AbstractClassDTO(String iri, String label) {}
```

DTOs are records when possible.

## 5. Mapper

```java
@Mapper(componentModel = "spring")
public interface AbstractClassMapper {
    AbstractClassDTO toDto(Class source);
    List<AbstractClassDTO> toDtoList(List<Class> source);
}
```

## 6. Controller

```java
@RestController
@RequestMapping("/api/datasets/{datasetId}/graphs/{graphId}/classes/abstract")
@RequiredArgsConstructor
public class AbstractClassRESTController {

    private final ListAbstractClassesUseCase useCase;
    private final AbstractClassMapper mapper;

    @GetMapping
    public Response<List<AbstractClassDTO>> list(@PathVariable String datasetId,
                                                 @PathVariable String graphId) {
        var graph = new GraphIdentifier(datasetId, graphId);
        return Response.ok(mapper.toDtoList(useCase.execute(graph)));
    }
}
```

## 7. Frontend API method

```js
// frontend/src/lib/api/backend.js (excerpt)
export const backend = {
  // …
  classes: {
    // …
    listAbstract: ({ datasetId, graphId }) =>
      fetchJson(`${base}/datasets/${datasetId}/graphs/${graphId}/classes/abstract`, {
        credentials: 'include',
      }),
  },
};
```

`fetchJson` is the project's existing wrapper that handles errors and JSON parsing.

## 8. Frontend usage

```svelte
<script>
  import { backend } from '$lib/api/backend.js';
  import { active } from '$lib/sharedState.svelte.js';

  let abstractClasses = $state([]);

  $effect(async () => {
    if (!active.dataset || !active.graph) return;
    abstractClasses = await backend.classes.listAbstract({
      datasetId: active.dataset.id,
      graphId: active.graph.id,
    });
  });
</script>

<ul>
  {#each abstractClasses as cls}
    <li>{cls.label}</li>
  {/each}
</ul>
```

## Tests

| Layer | Test |
| ----- | ---- |
| `ListAbstractClassesService` | In-memory `GraphPort`, Turtle fixture with two classes (one abstract). Assert the right one comes back. |
| `AbstractClassMapper` | Trivial unit test asserting fields are copied. |
| `AbstractClassRESTController` | MockMvc test mocking the use case. |
| Frontend | Optional Vitest test for the `backend.classes.listAbstract` URL composition. |

## Don'ts

- Don't put SPARQL strings inside Java code; load templates from the resources folder.
- Don't bypass `GraphPort` and reach into Jena directly.
- Don't add a parallel state store on the frontend; reuse `sharedState`.
- Don't construct backend URLs in components — go through `BackendConnection`.

## Pre-PR checklist

```bash
# Backend
cd backend
mvn -B spotless:apply
mvn -B verify

# Frontend
cd ../frontend
npm run format
npm run lint
npm run test
npm run build
```

If everything passes, your PR is ready. CI will rerun all of this on the cloud.
