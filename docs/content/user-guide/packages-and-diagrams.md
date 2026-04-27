---
title: Packages and Diagrams
sidebar_position: 7
---

# Packages and Diagrams

Packages structure the model; diagrams visualise one package at a time. Knowing how the two interact is the difference between fighting the editor and flowing through it.

## The package navigation tree

The left panel shows every package in the active graph as an alphabetically ordered tree. Each package node expands to reveal its classes.

![Add package](/img/screenshots/add-package.png)

From the tree you can:

- Click a package — the diagram view switches to it.
- Click a class — the class editor opens (the diagram stays put).
- Right-click a package — context actions (rename, delete, edit, new class, new package).
- Use the search bar to jump straight to a class without expanding the tree.

## Creating a package

From the package menu or the navigation tree's context menu choose **New package**. Provide:

- **Name** (used as IRI fragment and label).
- **Parent package** (optional — sub-packages are supported).
- **Comment**.

A new empty package shows up immediately in the tree.

## Editing a package

The package editor lets you rename, recomment, change the parent package, or delete a package. Renaming updates every class's `cims:belongsToCategory` reference; the changelog captures the rename atomically.

Deleting a package requires you to choose what happens to its classes:

- **Move them to another package** — recommended, non-destructive.
- **Delete them too** — only when you really mean it. The confirmation dialog spells out the consequences.

## The diagram view

The center pane is a UML-style diagram of the active package. Class boxes show:

- The class name and stereotype.
- A divider, then the attributes (name, datatype, multiplicity).
- Associations rendered as labelled edges to other class boxes — possibly to classes in other packages, in which case those boxes are shown faded.

![Add class](/img/screenshots/add-class.png)

### Interacting with the diagram

- **Pan** with click-and-drag on empty canvas, or with `Space` + drag.
- **Zoom** with the mouse wheel, the on-screen `+` / `−` buttons, or the keyboard shortcuts `Ctrl/Cmd + =` / `Ctrl/Cmd + −`. `Ctrl/Cmd + 0` resets zoom.
- **Move a class** by dragging its header. The new position is saved automatically.
- **Click a class** to focus it. **Double-click** to open the class editor.
- **Right-click** a class for the context menu (edit, copy IRI, focus parents, delete, …).
- **Right-click empty canvas** for the diagram context menu (new class, new association, layout reset).

### Filtering and view options

Open **Filter view** from the diagram toolbar to limit what the canvas shows:

- Hide attributes (compact view).
- Hide associations to external packages.
- Hide enumerations.
- Hide abstract classes.
- Show only classes whose name matches a substring.

These filters are local to the current browser session — they do not change the underlying model.

## Diagram layout

Layout is computed automatically the first time you open a package. After that, your manual adjustments are persisted as `arch:` triples in the graph, so:

- Re-opening the package later restores your layout.
- Exporting the schema (with layout) preserves it.
- Re-importing an exported schema brings the layout back.

If a layout becomes a mess, **Reset layout** in the diagram context menu re-runs the auto-layout from scratch. The change is recorded in the changelog and can be undone.

## Adding classes from the diagram

The diagram context menu has **New class**, which opens the same dialog as the navigation tree.

Drawing an association is a drag-from-class-to-class gesture: hover the source class until the connector handles appear, drag to the target class, and pick the multiplicity in the popup that appears.

## Cross-package context

Associations to classes in *other* packages are shown as connectors leaving the canvas, terminating at a faded "ghost" representation of the foreign class. Clicking the ghost takes you to the package that actually owns the foreign class.

## Performance notes

Very large packages (hundreds of classes) render slower. Two strategies help:

- Split a large package into sub-packages.
- Use the filter view to hide details you do not need.

If a package consistently feels slow, consider whether the level of nesting reflects the model intent or whether the package is functioning as a "miscellaneous" bucket that should be split.
