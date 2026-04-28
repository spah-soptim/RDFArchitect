---
title: Working with Namespaces
sidebar_position: 5
---

# Working with Namespaces

Namespaces are managed per dataset from **Edit → Manage Namespaces** (or **View Namespaces** when read-only). The dialog lists every prefix/URI pair currently defined for the dataset, lets you add new prefixes, rename them, or remove unused ones.

![Manage namespaces](/img/screenshots/manage-namespaces.png)

Prefix uniqueness is enforced: the save button stays disabled while two rows share a prefix, and the offending rows are highlighted. Namespaces used by any resource in the dataset cannot be deleted — the editor flags them as in-use.

Namespaces are also surfaced in every place where a URI is entered (new class dialog, attribute editor, import/export dialogs) so that you rarely need to type the full namespace by hand.
