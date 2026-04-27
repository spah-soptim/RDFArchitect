---
title: Contribution Scenarios
sidebar_position: 15
---

# Contribution Scenarios

Recipes for the most common contributions, with the exact files to touch and the gates that need to pass.

## Fix a UI bug

1. Reproduce locally (`npm run dev`).
2. Find the offending component under `frontend/src/`.
3. Make the smallest change that fixes the bug.
4. If the underlying logic is testable (validity rule, helper, model), add a Vitest test that demonstrates the bug and passes after the fix.
5. Run `npm run format && npm run lint && npm run test && npm run build`.
6. Open the PR; describe the bug and how to reproduce it.

Don't broaden scope: if the bug exposes an unrelated issue, file a separate ticket.

## Add a field to a class

End-to-end across both apps. Suppose you're adding `cims:isExperimental` (boolean):

1. **RDF conventions:** decide the predicate IRI; document it in [Data Model](./data-model).
2. **Backend domain model:** add the field to the `Class` domain object.
3. **Backend port:** extend the `findClass`/`saveClass` SPARQL templates (or whichever ones touch class triples) to read/write the new predicate.
4. **DTO:** add the field to `ClassDTO` and to the request DTO of `EditClassUseCase`.
5. **Mapper:** MapStruct will pick up the new field automatically when names match.
6. **SHACL generator:** decide whether the new field affects shape generation. If yes, extend `NodeShapeBuilder`.
7. **Backend tests:** add a fixture covering the new predicate. Assert read, write, round-trip, and SHACL.
8. **Frontend reactive wrapper:** add the field to `ReactiveClass`.
9. **Frontend editor:** add the input to the General tab of the class editor.
10. **Frontend tests:** Vitest test for `ReactiveClass.toPlain()` round-trip.
11. **Docs:** mention the new field in [Editing Classes](/user-guide/editing-classes).

## Fix a SHACL generation bug

1. Find a model that triggers the bug. Encode it as a Turtle fixture under `backend/src/test/resources/testdata/shacl/`.
2. Write a failing test in the SHACL generator package that asserts the *correct* output.
3. Fix the generator; the test should now pass.
4. Confirm no other generator tests broke (deterministic generation means a regression is detectable as a diff).
5. Update the [Data Model](./data-model) page if the predicate semantics changed.

## Add a migration pattern

Suppose CGMES adds a new "domain renamed with namespace change" pattern:

1. Add a SPARQL template under `backend/src/main/resources/sparql-templates/migration/`.
2. Add a `MigrationStep` enum value if needed.
3. Wire the new step into the composer.
4. Surface it in the wizard's UI — typically as a new row in the relevant existing step rather than a sixth step.
5. Add a backend integration test running the composer on a synthetic comparison and asserting the produced SPARQL.
6. Document the new pattern in [Migration Wizard](/user-guide/migration-wizard).

## Documentation-only change

1. Edit the relevant Markdown under `docs/content/`.
2. Run the local Docusaurus dev server (`cd docs && npm start`).
3. Verify rendering and intra-page links.
4. Open the PR with title `docs: …`.

The `deploy-docs.yml` workflow builds the site as a verification step; on merge to main the site is published.

## Bump a dependency manually

When Renovate is wrong or slow, you can bump manually. See [Dependencies](./dependencies) for the regenerate-license-files dance.

## Add a new endpoint

Walk through [Adding a Feature End-to-End](./adding-a-feature) — that page is the canonical tour.

## Add a new locale (future)

Not supported in 1.0. If/when i18n lands, this section will document it. Don't bolt on per-component translation tables in the meantime; they would create a parallel structure that doesn't go anywhere.

## Don'ts (cross-cutting)

- Don't refactor adjacent code "while you're there". Keep the PR scope tight.
- Don't introduce a new state library, HTTP client, or styling system.
- Don't broaden the public REST API "just in case".
- Don't bypass the changelog when writing graph mutations.
- Don't skip tests because the change is small. Small changes are exactly the changes that introduce regressions.
