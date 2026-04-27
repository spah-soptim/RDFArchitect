---
title: Read-Only Mode
sidebar_position: 14
---

# Read-Only Mode

A graph or an entire dataset can be marked **read-only**. Read-only is the right state for any model that is "blessed" or shared with reviewers who shouldn't be able to mutate it accidentally.

## Where read-only is set

| Scope | Where to toggle | Affects |
| ----- | --------------- | ------- |
| **Snapshot** | Always read-only. Cannot be made writable. | The snapshot itself (the source graph is unaffected). |
| **Graph** | Graph menu → **Read-only**. | All editing of that graph in any dataset. |
| **Dataset** | Dataset menu → **Read-only**. | Every graph in the dataset (including new graphs you might create). |
| **Application-wide** | Backend config (administrator). | All datasets — used for staging and demo deployments. |

The toolbar shows an unmistakeable read-only badge whenever you are looking at read-only content.

## What changes when read-only is on

| Feature | Read-only graph |
| ------- | --------------- |
| Browsing the model | ✅ Works |
| Diagram navigation | ✅ Works |
| Class / package / SHACL inspection | ✅ Works |
| Comparison | ✅ Works |
| Export | ✅ Works |
| Snapshot creation | ✅ Works (a snapshot of a read-only graph is fine) |
| Importing into the graph | ❌ Disabled |
| Creating / editing / deleting classes, packages, attributes, etc. | ❌ Disabled |
| Undo / redo / restore | ❌ Disabled (history view stays browsable) |
| Migration wizard runs | ❌ Disabled when the *target* is read-only |

Editing affordances aren't merely greyed out — they're hidden, so the UI looks like a clean reading surface rather than a disabled editor.

## How the toggle behaves

Toggling read-only is itself a recorded action (in the changelog). If you flip a graph back to writable, every editing affordance returns immediately.

## Why use it?

- **Reviewer-friendly handover.** Send a read-only graph (or its snapshot) to colleagues without worrying about accidental edits.
- **Stable references.** Lock a graph that other graphs compare against.
- **Demo and training.** A read-only dataset on a shared host gives newcomers something to explore without a risk of breaking things.

## Combining read-only with snapshots

Snapshots are always read-only. Marking the source graph read-only is a different decision — useful when you want to lock the working copy too, after declaring a milestone reached.
