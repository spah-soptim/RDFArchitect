---
title: Snapshots and Sharing
sidebar_position: 10
---

# Snapshots and Sharing

A **snapshot** is a frozen, read-only copy of a graph. Snapshots are how you hand a schema to a reviewer who shouldn't (or doesn't want to) edit it.

![Share snapshot](/img/screenshots/share-snapshot.png)

## Creating a snapshot

From the graph menu choose **Snapshot → New snapshot**. Provide:

- **Name** — a short label (e.g. `cgmes-3-rc1`, `2026-04-25-review`).
- **Description** — optional context for reviewers.

When you confirm, RDFArchitect copies the current state of the graph into a separate read-only named graph. The operation is fast (just a graph copy) and produces a permalink you can share.

## Sharing the link

The snapshot URL is shown right after creation and is also listed under **Snapshot → Manage snapshots**. Anyone who can reach your RDFArchitect instance and has the URL can browse the snapshot.

If your deployment requires authentication (it should — see [Access control](/admin-guide/access-control)), reviewers need credentials too. RDFArchitect itself does not gate the link.

## What reviewers see

Following a snapshot link opens the editor in read-only mode pinned to that snapshot. Reviewers can:

- Browse the package tree.
- Open class details (read-only).
- View the diagram.
- Inspect SHACL.
- Compare the snapshot to other graphs / snapshots / files.

Reviewers cannot edit, import, undo, restore, or delete anything.

## Listing and deleting snapshots

**Snapshot → Manage snapshots** lists every snapshot in the active graph, with creation time, author, and a link to the snapshot view. From there you can delete snapshots you no longer need.

Deletion is permanent. The original graph is untouched — only the frozen copy goes away.

## Comparing against a snapshot

The comparison view treats snapshots the same as live graphs, so you can:

- Compare your in-progress working graph against the last review snapshot to see what's changed.
- Compare two snapshots to audit the delta between two milestones.

See [Comparing schemas](./comparing-schemas).

## Exporting a snapshot

Snapshots can be exported just like any read-only graph. Use **Export** while viewing the snapshot.

## When to take a snapshot

Good moments to take one:

- Before a risky bulk edit (lets you compare and roll back via comparison).
- At the end of a release-readiness review.
- Before kicking off a migration wizard run.
- Whenever you want to send a colleague a stable URL that won't change as you keep editing.

Snapshots are cheap. There is no penalty for taking many.

## Limitations in 1.0

- Snapshots cannot be re-promoted to working graphs in-place. If you need that, export the snapshot and import it as a new graph.
- There is no built-in commenting or annotation surface on snapshots; reviewers comment via your usual channels (mail, ticket, chat).
- Bulk-deleting many snapshots requires deleting them one at a time.
