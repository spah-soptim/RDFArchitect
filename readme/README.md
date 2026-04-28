# RDFArchitect Documentation

Documentation for **RDFArchitect 1.0.0** — the open-source web-based editor for CIM/CGMES and ENTSO-E RDFS schemas with SHACL constraints.

## Who should read what

If you are new to RDFArchitect, start with **[Introduction](#introduction)** below, then move on to the **User Guide**. Otherwise, the table below points you to the most relevant document for your role.

| If you are…                                 | Start with                                    | Then read                                                          |
| ------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------ |
| An **electrical engineer** / CIM modeller   | [User Guide](user-guide.md)                    | [CIM / CGMES concepts mapping](cim-mapping.md), [Features](features.md) |
| A **software architect** / schema maintainer | [User Guide](user-guide.md), section 10–12    | [Features](features.md), [CIM mapping](cim-mapping.md)             |
| A **project manager** / product owner       | [Features](features.md)                        | [User Guide](user-guide.md), sections 9–12                         |
| An **administrator** deploying the service  | [Installation](installation.md)                | [Administrator's guide](administrator-guide.md), [FAQ](faq.md)     |
| A **developer** contributing or extending   | [Developer guide](developer-guide.md)          | [`.github/CONTRIBUTING.md`](../.github/CONTRIBUTING.md)            |
| Just looking for screenshots                | [Screenshots](screenshots.md)                  |                                                                    |

---

## Introduction

RDFArchitect is a web-based tool for **visualising, editing, and sharing RDFS schemas with CIM extensions** — as used in CGMES and the ENTSO-E Network Code Profiles — together with their **SHACL constraints**. It is an open-source alternative to proprietary CIM/RDF modelling tools.

### What you can do with it

- **Import and export** CGMES and ENTSO-E profiles in RDF/XML, Turtle, or N-Triples.
- **Visualise** classes, attributes, associations, and inheritance as UML-style diagrams, organised by package.
- **Edit** every aspect of a profile — classes, attributes, associations, enum entries, stereotypes, comments, namespaces, and the profile header — through a validating UI.
- **Generate SHACL** automatically from the schema, and manage imported SHACL alongside it.
- **Track changes** with an edit changelog, undo/redo, and the ability to restore any previous state.
- **Compare** two versions of a profile and see a structured list of additions, removals, and modifications.
- **Migrate** instance data between profile versions by generating a reviewable **SPARQL UPDATE script** through a guided wizard.
- **Share** a complete, read-only, navigable view of a profile via a single URL — no downloads, no tooling required by the recipient.

### Architecture, briefly

- **Frontend** — SvelteKit single-page application
- **Backend** — Spring Boot REST service
- **Triple store** — Apache Jena Fuseki (any SPARQL 1.1 endpoint works)

Everything is open-source, Apache 2.0 licensed. See [`LICENSE`](../LICENSE) for the full text.

---

## Documents in this folder

- **[User Guide](user-guide.md)** — the full walkthrough: concepts, workspace, import, editing, SHACL, compare, migration, sharing.
- **[Features reference](features.md)** — feature checklist organised by role, plus a where-in-the-UI quick reference.
- **[Installation](installation.md)** — Docker Compose and local development setup, configuration reference, production deployment notes.
- **[Administrator's guide](administrator-guide.md)** — for the operator: triple store, backups, access control, upgrades, monitoring.
- **[Developer guide](developer-guide.md)** — repository layout, backend and frontend architecture, end-to-end feature walkthrough, testing, code style, release process.
- **[CIM / CGMES concepts mapping](cim-mapping.md)** — how RDFArchitect terminology relates to CIM, CGMES, and ENTSO-E.
- **[FAQ & troubleshooting](faq.md)** — common questions and fixes.
- **[Screenshots](screenshots.md)** — visual overview of the editor.

---

## Community and support

- **Bugs and feature requests**: [GitHub Issues](https://github.com/SOPTIM/RDFArchitect/issues)
- **Questions and discussion**: [GitHub Discussions](https://github.com/SOPTIM/RDFArchitect/discussions)
- **Security reports**: see [`SECURITY.md`](../.github/SECURITY.md)
- **Contributing**: see [`CONTRIBUTING.md`](../.github/CONTRIBUTING.md)
- **Commercial support**: SOPTIM AG — see the [SOPTIM GitHub organisation](https://github.com/SOPTIM) for contact information

---

*Version: 1.0.0 — Released 2026-04-24.*
*See [`CHANGELOG.md`](../CHANGELOG.md) for the release history.*
