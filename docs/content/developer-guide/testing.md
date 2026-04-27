---
title: Testing
sidebar_position: 10
---

# Testing

The backend has a high test bar; the frontend is more pragmatic. Both run on every PR.

## Backend testing patterns

### Unit tests

Lightweight JUnit 5 + Mockito + AssertJ. Use them for thin delegations — controllers, mappers, helpers.

```java
@ExtendWith(MockitoExtension.class)
class ClassRESTControllerTest {

    @Mock ListClassesUseCase listClasses;
    @Mock ClassMapper mapper;
    @InjectMocks ClassRESTController controller;

    @Test
    void delegatesToUseCase() {
        var graph = new GraphIdentifier("default", "g1");
        when(listClasses.execute(graph)).thenReturn(List.of(/* … */));

        var response = controller.list("default", "g1");

        assertThat(response.body()).hasSize(1);
        verify(listClasses).execute(graph);
    }
}
```

### Service tests

For business logic, use the in-memory database adapter and Turtle fixtures:

```java
@SpringBootTest
class EditClassServiceTest {

    @Autowired EditClassUseCase editClass;
    @Autowired GraphPort graphPort;

    @BeforeEach
    void setup() {
        graphPort.loadFixture("testdata/single-class.ttl");
    }

    @Test
    void editsLabel() {
        editClass.execute(graph, "cim:Breaker", new ClassEdit().withLabel("Circuit breaker"));

        try (var txn = graphPort.beginRead(graph)) {
            assertThat(txn.findClass("cim:Breaker").label()).isEqualTo("Circuit breaker");
        }
    }
}
```

Fixtures live in `backend/src/test/resources/testdata/`. Always keep them as small as possible — one or two classes is usually enough.

### Mapper tests

Trivial unit tests asserting field copy. MapStruct generates the implementation; you're testing your mapping spec, not generated code.

### Graph behaviour tests

For graph operations, assert the final RDF state:

```java
assertThat(graphPort.exportTurtle(graph))
    .contains("rdfs:label \"Circuit breaker\"");
```

Combine with explicit assertions on the changelog and (where relevant) layout side effects.

### Coverage

The default Maven build runs JaCoCo and fails below the configured threshold. Aim for behaviour coverage over line coverage — a test that exercises a real graph mutation through `mvn verify` is worth more than three mocked unit tests.

## Frontend testing patterns

### Vitest

```bash
cd frontend
npm test
```

Pragmatic targets:

- Reactive wrappers (`ReactiveClass`, `ReactiveAttribute`, …).
- Validity rules (pure functions).
- Helpers in `lib/utils/` and `lib/scripts/`.
- Backend connection URL composition.

Components are tested through the helpers they depend on. Visual / route-composition changes are verified via `npm run build` and manual exercising.

### Manual checklist in PR template

The PR template asks contributors to confirm a manual checklist (open the editor, import a file, edit a class, view SHACL, …). For UI changes, run through it before requesting review.

## Pre-PR command sequence

The full reproducible gate, identical to CI:

```bash
# Backend
cd backend
mvn -B spotless:apply
mvn -B -Plint -DskipTests verify
mvn -B verify

# Frontend
cd ../frontend
npm run clean-install
npm run format
npm run lint
npm run test
npm run build

# Docs (only if you touched docs/)
cd ../docs
npm install
npm run build
```

## What CI checks

- Backend: `mvn verify` (Spotless, Checkstyle, SpotBugs, JaCoCo, JUnit), license file freshness.
- Frontend: lint, test, build, license file freshness.
- Docs: Docusaurus build.
- PR title: Conventional Commits format.
- DCO sign-off on every commit.

## When tests are flaky

Flakes are bugs. Don't rerun until green; fix the root cause:

- Concurrency — most flakes here come from missing `commit()` or premature transaction end.
- Time-dependent assertions — use the project's `Clock` indirection.
- Order-dependent assertions on `Set`-typed results — sort explicitly.

Mark a flaky test with `@Disabled` only as a last resort and only with a linked tracking issue.
