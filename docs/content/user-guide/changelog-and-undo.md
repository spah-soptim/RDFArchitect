---
title: Changelog and Undo
sidebar_position: 9
---

# Changelog and Undo

Every write operation in RDFArchitect is recorded. You can review history, undo recent steps, or roll an entire graph back to an earlier point in time.

![Changelog](/img/screenshots/changelog.png)

## Undo and redo

The toolbar carries the standard pair of arrows. Their behaviour:

- **Undo** (`Ctrl/Cmd + Z`) — reverts the most recent operation on the active graph.
- **Redo** (`Ctrl/Cmd + Shift + Z`) — reapplies the operation that was just undone.

Operations are atomic: a single user action (e.g. "save class") is one undo step, even if it touches many triples internally.

The undo stack is per-graph. Switching graphs does not clear it.

## The changelog view

Open the changelog from the toolbar (or via the keyboard shortcut `Ctrl/Cmd + H`). The page lists every change to the active graph in reverse chronological order. Each entry shows:

| Column | Meaning |
| ------ | ------- |
| **Time** | Wall-clock time of the change. |
| **Action** | Short summary (e.g. *Added attribute*, *Renamed class*). |
| **Target** | The class, attribute, package, or shape affected. |
| **Details** | Click to expand into before/after triples. |

Filters at the top of the list narrow by type of action, by target, or by time range.

## Inspecting a change

Expanding an entry reveals the exact triples added and removed:

```turtle
# Removed
ex:Breaker a rdfs:Class ; rdfs:label "Breaker" .

# Added
ex:Breaker a rdfs:Class ; rdfs:label "Circuit breaker" .
```

This view is also the inline-diff format used in the comparison feature ([Comparing schemas](./comparing-schemas)).

## Restoring an earlier state

Each changelog entry has a **Restore** action. Choosing it asks you to confirm and then reverts the entire graph to the state immediately *before* that entry. Three things to know:

1. Restoring is itself recorded in the changelog. You never lose history.
2. Restoring after a long sequence of changes can be a much larger operation than a single undo — review the diff before confirming.
3. If the graph has snapshots, restoring does not affect them.

## Author tracking

If your reverse proxy injects an authenticated user (via `X-Authenticated-User` or similar), changelog entries are tagged with that identity. Otherwise entries are anonymous. See [Access control](/admin-guide/access-control) for setup.

## Retention

By default the changelog retains every entry indefinitely. Administrators can prune old entries via the Fuseki admin tools — see [Backups](/admin-guide/backups). Pruning is irreversible, so retain a Fuseki backup before doing it.

## What is and isn't tracked

| Tracked | Not tracked |
| ------- | ----------- |
| Class / package / attribute / association / enum-entry CRUD. | Diagram-only zoom and pan state. |
| Custom SHACL imports. | Filter view selections (session-local). |
| Renames, including ripple updates to associations. | Read-only mode toggles. |
| Layout changes (when persisted to the graph). | Snapshot creation (recorded separately). |
| Graph-level imports (replace and merge). | Login / logout events (handled by your proxy). |
| Restore-to operations. | |
