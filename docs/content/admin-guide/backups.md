---
title: Backups and Restore
sidebar_position: 6
---

# Backups and Restore

Everything that matters lives in Fuseki. The backup story is a Fuseki backup story.

## What to back up

| Item | Where it lives | Frequency |
| ---- | -------------- | --------- |
| **Triple store data** | Fuseki's TDB2 database directory. | Daily (minimum), more often for active editing teams. |
| **Application config** | `/etc/rdfarchitect/` (or your equivalent). | Whenever it changes; under version control. |
| **Reverse-proxy config** | Wherever you keep it. | Whenever it changes; under version control. |
| **Fuseki admin config** | Fuseki's `configuration/` directory. | Whenever it changes. |

You do **not** need to back up:

- The application JAR / Docker images — they're reproducible from tags.
- The frontend static bundle — same.
- Logs — they're useful but not part of recovery.

## Fuseki backup options

### Option 1 — Filesystem snapshot (simplest)

Stop Fuseki, snapshot the volume, restart. Best with copy-on-write filesystems (ZFS, btrfs, EBS snapshots).

```bash
docker stop fuseki
zfs snapshot tank/fuseki@$(date +%F)
docker start fuseki
```

For a typical schema-modeling deployment the downtime is seconds.

### Option 2 — Fuseki backup endpoint

Fuseki ships with `/$/backup/{dataset}` which writes a compressed `.nq.gz` to the `backups/` directory:

```bash
curl -u admin:changeme \
  -X POST http://fuseki:3030/$/backup/production
```

Schedule via cron. The backup is consistent — Fuseki uses a snapshot read transaction internally.

### Option 3 — `tdb2.tdbdump`

Offline dump using the TDB2 CLI. Requires Fuseki to be stopped. Useful for migrations, not for routine backups.

## Restore

### From a filesystem snapshot

Stop Fuseki, restore the volume, start Fuseki. Verify by hitting `/$/datasets`.

### From a `.nq.gz`

```bash
curl -u admin:changeme \
  -X POST -H "Content-Type: application/n-quads" \
  --data-binary @production-backup.nq.gz \
  --header "Content-Encoding: gzip" \
  http://fuseki:3030/production/data
```

Or stop Fuseki and use `tdb2.tdbloader2`.

### Test restores

A backup you haven't tested is not a backup. Quarterly:

1. Restore the latest backup into a fresh Fuseki container.
2. Point a test RDFArchitect against it.
3. Verify a representative dataset opens, classes render, SHACL displays.

Bake this into your operational calendar.

## Per-graph "soft" backups

Two in-app mechanisms are useful complements (but not replacements) for Fuseki backups:

- **Snapshots.** Take a snapshot before any risky bulk edit; the snapshot is preserved as a separate named graph and can be restored via export/import if needed.
- **Changelog restore.** Restoring a graph to an earlier changelog entry is non-destructive (the restoration is itself a changelog entry).

Both rely on Fuseki being intact. If Fuseki is gone, snapshots and changelog are gone.

## Retention

Recommended starting policy:

- Daily backups for 14 days.
- Weekly backups for 12 weeks.
- Monthly backups for 12 months.

Adjust to your compliance regime. Compress and store off-site.

## Encryption

Backups should be encrypted at rest (use your storage layer's encryption — S3 SSE, ZFS native encryption, LUKS, …). RDFArchitect does not encrypt application-level secrets at rest because there are no application-level secrets.
