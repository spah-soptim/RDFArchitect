---
title: Prefixes and Namespaces
sidebar_position: 13
---

# Prefixes and Namespaces

Every RDF resource lives in a namespace, and every namespace has a short prefix used to write IRIs concisely. RDFArchitect manages prefixes globally and per dataset.

![Manage namespaces](/img/screenshots/manage-namespaces.png)

## Where prefixes show up

- In the class IRI field (`cim:Breaker`).
- In the attribute datatype drop-down (`xsd:string`).
- In the SHACL inspection views.
- In every export.

When a prefix is missing, IRIs are shown in their full form. That's a sign you should add the prefix.

## Levels of configuration

| Level | Scope | Where set | Used when |
| ----- | ----- | --------- | --------- |
| **Global** | All datasets, system-wide. | Application config / admin UI. | Sensible defaults that almost everyone needs (`rdf`, `rdfs`, `owl`, `xsd`, `sh`, `cim`). |
| **Dataset** | One dataset, all its graphs. | Dataset menu → **Manage namespaces**. | Project-specific prefixes (e.g. `entso-e`, your in-house extension). |

Dataset prefixes override global prefixes with the same short name.

## The prefix manager

**Manage namespaces** lists every prefix currently active in the dataset. Columns:

| Column | Description |
| ------ | ----------- |
| **Prefix** | Short name (e.g. `cim`). |
| **Namespace IRI** | Full URI (e.g. `http://iec.ch/TC57/CIM100#`). |
| **Source** | `global` (read-only here) or `dataset`. |

You can:

- **Add** a new prefix. Both fields are required and must be unique within the dataset.
- **Edit** a dataset-level prefix (override the global definition or rename).
- **Remove** a dataset-level prefix (falls back to the global definition, if any).

Changes take effect immediately for new operations; already-rendered diagrams may need a refresh.

## Property prefixes

Property prefixes (`cim:`, `cims:`) come pre-installed and are tightly tied to CIM tooling. You usually don't change those — they exist because export to ENTSO-E-compatible files expects them.

## What gets imported

When you import a Turtle file, every prefix declaration in the file is added to the dataset's prefix list (without overwriting existing dataset-level prefixes). This is what makes round-tripping a foreign file pleasant: re-exports come back with the same prefixes the file came in with.

## Common pitfalls

- **Two prefixes for the same namespace** — usually the result of importing files from two upstream sources. The prefix manager flags duplicates so you can pick one and remove the other.
- **A namespace IRI without a trailing `#` or `/`** — RDF concatenates the local name onto the namespace verbatim. Make sure your namespace ends with the separator the file expects.
- **Prefix used inside literal values** — prefixes only expand inside IRIs. A string `"cim:foo"` stays a string.

## Limitations

- The global prefix list is configured at deployment time; end users cannot edit it from the UI in 1.0.
- A prefix change does not rewrite triples — it only changes how they are displayed and serialized.
