---
title: Screenshots
sidebar_position: 15
---

# Screenshot Gallery

A visual tour of the editor and the main workflows.

## Welcome page

![Welcome page](/img/screenshots/homepage.png)

The entry point. List of datasets, a quick-create button, links to documentation and to the active graph.

## Editor overview

![Editor overview](/img/screenshots/editor.png)

Left: package and class navigation. Center: package-focused diagram. Right: contextual actions. The active dataset/graph indicator and the toolbar are pinned at the top.

## Importing a schema

![Import schema dialog](/img/screenshots/import-schema.png)

Pick the destination dataset and graph, drop the file, and confirm. The dialog detects the serialization from the file extension and surfaces SHACL shapes inside the file as custom shapes after import.

## Managing namespaces

![Manage namespaces](/img/screenshots/manage-namespaces.png)

Per-dataset namespace and prefix configuration. Global prefixes are visible but cannot be edited here.

## Adding a package

![Add package](/img/screenshots/add-package.png)

Packages can be created directly from the navigation tree. Provide a name, optional parent package, and an optional comment.

## Adding a class

![Add class](/img/screenshots/add-class.png)

The new-class dialog is contextual: dataset, graph, namespace and package are pre-filled based on where you opened it from.

## Class editor

![Class editor](/img/screenshots/class-editor.png)

The main editing surface. Tabs for general metadata, attributes, associations, enum entries (when applicable), and SHACL.

## SHACL inspection

![SHACL view](/img/screenshots/shacl.png)

Generated and custom shapes are listed side by side, clearly distinguished. Property shapes can be drilled into for the full constraint detail.

## Changelog

![Changelog](/img/screenshots/changelog.png)

Per-graph history. Each entry expands to a triple-level diff and offers a *Restore* action.

## Schema comparison

![Compare schemas](/img/screenshots/compare.png)

Side-by-side comparison of two graphs, snapshots, or files. Differences are organised at package, class, and property level with inline diffs.

## Snapshots and sharing

![Share snapshot](/img/screenshots/share-snapshot.png)

Snapshots produce a stable, read-only URL that reviewers can browse without affecting the working graph.
