---
title: Schema Migration Wizard
sidebar_position: 12
---

# Schema Migration Wizard

The migration wizard helps you produce a SPARQL Update script that brings instance data from one schema version to another. It walks through five guided steps and emits a transparent, auditable artefact you can run against your own triple store.

## When to use it

Use the wizard when you need to evolve instance data along with a schema, for example:

- Migrating CGMES instance files from CIM 16 to CIM 17.
- Renaming a property family across an entire dataset.
- Removing deprecated classes and re-homing their data on a successor.

Use plain export-and-reimport when you only care about the schema, not about pre-existing instance data.

## The five steps

### 1. Select source and target schemas

Pick the *from* and *to* schemas. Both can be live graphs, snapshots, or uploaded files. The wizard runs the comparison engine against the pair so the next steps know what changed.

The wizard offers candidate operations based on the comparison: classes that were renamed, properties that changed datatype, enum entries that were removed, etc. You always get the final say in steps 2–4.

### 2. Confirm class renames

A list of detected class renames, with confidence scores. For each candidate:

- **Accept** the suggested rename.
- **Override** the target class manually.
- **Mark as unrelated** (the wizard treats source and target as independent classes).

You can also add renames that the comparison missed.

### 3. Confirm property renames

Same process for attribute and association renames. The list is grouped by source class, so you can step through one class at a time. Each row shows old/new datatype and old/new cardinality if those changed too.

### 4. Confirm defaults for new mandatory properties

If the target schema introduces a property with `multiplicity ≥ 1` that the source schema didn't have, instance data won't validate without a value. The wizard asks you to provide a default — a literal, a SPARQL expression, or "skip and let validation flag it later".

### 5. Export migration script

The final step produces a SPARQL Update script (a sequence of `DELETE/INSERT` blocks) plus an explanatory README. You can:

- Download the SPARQL file.
- Copy it to the clipboard.
- Run it against an external Fuseki — the wizard does *not* mutate any RDFArchitect graph automatically.

The script is composed from the SPARQL templates shipped with RDFArchitect (you can see them in the source under `backend/src/main/resources/sparql-templates/migration/`). Each block is preceded by a comment explaining what it does, so the file is auditable without running it.

## Example output

```sparql
# RDFA-generated migration script
# Source: cim-16-equipment   Target: cim-17-equipment
# Generated 2026-04-25

# --- Class rename: Breaker -> CircuitBreaker ---
DELETE { ?s a cim16:Breaker }
INSERT { ?s a cim17:CircuitBreaker }
WHERE  { ?s a cim16:Breaker } ;

# --- Attribute rename: name -> displayName on Terminal ---
DELETE { ?t cim16:name ?v }
INSERT { ?t cim17:displayName ?v }
WHERE  { ?t cim16:name ?v ; a cim17:Terminal } ;
```

## Running the script

Recommended sequence:

1. Take a backup or snapshot of the data you're about to migrate.
2. Run the script in a non-production environment first.
3. Validate the result with the target schema's SHACL shapes.
4. If validation passes, replay against production.

The wizard intentionally stops short of pressing the button itself — running the script is a deliberate, auditable act.

## Limitations in 1.0

- The wizard generates SPARQL Update — it does not handle CSV/Excel/JSON sources natively.
- Cardinality reductions (e.g. `0..*` → `0..1`) are flagged but not auto-resolved; you decide which value to keep.
- Cross-graph rewrites (instance graphs that span multiple named graphs) require manual stitching of the produced script.
- "Confidence scores" are heuristics; review every suggestion before accepting.
