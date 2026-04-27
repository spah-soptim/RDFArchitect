---
title: Apache Jena Fuseki
sidebar_position: 4
---

# Apache Jena Fuseki

Fuseki is the triple store RDFArchitect persists everything to. Treat it as the system of record.

## Why Fuseki

- It is the reference SPARQL 1.1 server from the Apache Jena project.
- It is open source under Apache 2.0.
- It supports the full RDF dataset model (named graphs, transactions, SHACL).
- It is well-understood operationally (CLI tools, backup formats, file-based storage).

RDFArchitect uses standard SPARQL queries and the Graph Store Protocol — no Fuseki-specific extensions. In principle any SPARQL 1.1 endpoint that supports named graphs works, but Fuseki is the only one tested.

## Provisioning

### Container

```bash
docker run -d --name fuseki \
  -p 3030:3030 \
  -e ADMIN_PASSWORD=changeme \
  -v /srv/fuseki-data:/fuseki/databases \
  --restart unless-stopped \
  stain/jena-fuseki:5
```

The volume at `/fuseki/databases` is your data. Back it up.

### Bare metal

Download from the [Apache Jena releases page](https://jena.apache.org/download/), set `FUSEKI_BASE` to a writable directory, and run `fuseki-server`. systemd unit recommended.

## Storage choice

Fuseki supports two on-disk persistent backends:

| Backend | Use |
| ------- | --- |
| **TDB2** | Default. Multi-version concurrency, transactional, recommended. |
| **TDB1** | Legacy. Don't use for new installs. |

RDFArchitect doesn't care which you pick — it talks SPARQL to whatever you provide.

## Datasets

A Fuseki dataset is what RDFArchitect calls a *dataset*. Each one needs to be created in Fuseki before RDFArchitect can use it.

### Creating a dataset

Through the Fuseki admin UI (`http://fuseki:3030/`) → Datasets → New, choose "Persistent (TDB2)", give it a name. Or via the HTTP API:

```bash
curl -u admin:changeme \
  -F "dbName=production" \
  -F "dbType=tdb2" \
  http://fuseki:3030/$/datasets
```

RDFArchitect will create datasets via its own UI when authorised; behind the scenes, those calls go to the Fuseki admin API. Your service account needs admin privileges on Fuseki.

### `defaultDataset`

The backend property `database.defaultDataset` specifies which dataset is created (and used) on first start if none exist. In a fresh deployment, name it after your environment (`development`, `staging`, `production`).

## Fuseki authentication

Fuseki has its own basic-auth user database. RDFArchitect connects with one service account; end users never authenticate to Fuseki directly.

Recommended setup:

1. Create a Fuseki admin user (for ops).
2. Create a service user with read/write on all relevant datasets, used by the RDFArchitect backend.
3. Pass the service credentials to RDFArchitect via the connection URL or environment variables (Spring's standard HTTP basic auth properties on the Jena client).

### Network exposure

Fuseki must **not** be reachable from the public internet. Bind it to the internal network only. The reverse proxy in front of RDFArchitect should not proxy `/$/...` Fuseki admin paths.

## Snapshots and named graphs in Fuseki

RDFArchitect snapshots are stored as additional named graphs under a reserved IRI prefix (`arch:snapshots/...`). They show up in Fuseki's named-graph listings just like working graphs. From Fuseki's perspective they are normal data — back them up and replicate them like everything else.

## Performance tuning

For typical CIM schemas (hundreds of classes, thousands of triples), Fuseki is fast on default settings. Consider tuning when:

- Graphs exceed 1M triples — bump JVM heap (`-Xmx`), enable TDB2 statistics.
- Many concurrent writers — RDFArchitect serialises writes per-graph, so this rarely happens; if it does, reach out via GitHub Discussions.

## Upgrading Fuseki

Fuseki upgrades are independent of RDFArchitect upgrades. The TDB2 format is stable across Fuseki 4.x and 5.x. Always back up first; test in a non-production environment.
