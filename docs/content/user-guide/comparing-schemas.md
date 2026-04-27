---
title: Comparing Schemas
sidebar_position: 11
---

# Comparing Schemas

The comparison view answers the question "what changed between these two schemas?" — at the package, class, and property level, with an inline diff for individual triples.

![Compare schemas](/img/screenshots/compare.png)

## Opening the comparison

From the toolbar choose **Compare**. The dialog asks you to pick a *left* and a *right* side. Either side can be:

- A live graph in the current dataset.
- A snapshot of any graph.
- A file you upload from disk.
- A graph in a different dataset (if your installation has multiple).

The comparison runs server-side; large schemas may take a few seconds.

## Reading the result

The result page is split into sections, each expandable:

| Section | What it lists |
| ------- | ------------- |
| **Packages** | Packages added on the right, removed from the left, or renamed. |
| **Classes** | Per-package, the classes added/removed/changed. |
| **Attributes** | Per-class, the attributes added/removed/changed (datatype, cardinality, fixed value, etc.). |
| **Associations** | Per-class, association deltas including renamed targets. |
| **Enum entries** | For enumerations, the entries added/removed/renamed. |
| **SHACL shapes** | Constraint shapes that differ — both generated (because the model differs) and custom (because the file differs). |
| **Other triples** | Anything that doesn't fall into the categories above. |

Each row can be expanded to show the inline triple-level diff.

## Inline diffs

Inline diffs show:

```diff
- <removed triple>
+ <added triple>
```

Whitespace differences inside `rdfs:comment` literals are normalized so that re-flowed comments don't drown the real change.

## Filtering

The result page has filter chips:

- **Hide additions** / **Hide removals** / **Hide changes**.
- **Only show classes** / **Only show SHACL** / **Only show enums**.
- **Only show package X**.

These filters are local; they don't affect what's recorded in the changelog.

## Use cases

- **Pre-merge review.** Compare your working graph to the blessed baseline before importing the latter back as the new working graph.
- **Release planning.** Compare two snapshots that flank a release window.
- **Migration planning.** Compare the source and target schemas before launching the [migration wizard](./migration-wizard).
- **External alignment.** Compare your in-house extension against the upstream CIM/CGMES file you started from.

## Exporting the comparison

The comparison page can be exported as:

- **HTML** — same view, suitable to attach to a review ticket.
- **Turtle diff** — the actual changed triples as two Turtle blocks (added and removed).

## Limitations

- The compare engine does its best to detect renames (same triples on a class with a new IRI), but a rename combined with structural changes can still appear as a delete-plus-add pair. Inspect manually if it looks suspicious.
- Comparing two very large schemas may exceed the default request timeout. If that happens, narrow the scope by comparing one package at a time.
