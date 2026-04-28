---
title: Read-Only Mode
sidebar_position: 12
---

# Read-Only Mode

Every dataset has a read-only flag. When the flag is set, all editing actions are disabled in the UI, menu entries switch to their **View** variants, and the save buttons in every dialog are hidden. This is used for two purposes:

- **Protect a dataset** that represents an official, released profile from accidental changes. Use **Edit → Disable Editing** (shown when editing is currently enabled) to lock a dataset.
- **Presentation mode** — the read-only state applied to a snapshot is what makes the shared link safe to send out.

To resume editing, use **Edit → Enable Editing**. Enabling editing does not silently revert anything — it only lifts the write-protection.
