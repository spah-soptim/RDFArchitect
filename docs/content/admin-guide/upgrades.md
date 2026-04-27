---
title: Upgrades
sidebar_position: 7
---

# Upgrades

Procedure for moving RDFArchitect from one version to another.

## General procedure

1. **Read the changelog.** [`CHANGELOG.md`](https://github.com/SOPTIM/RDFArchitect/blob/main/CHANGELOG.md) lists every change. Pay particular attention to the `### Breaking Changes` headings.
2. **Check the version compatibility matrix below.**
3. **Back up Fuseki.** Take a fresh backup before any upgrade.
4. **Test in staging.** Restore the production backup into a staging Fuseki, point a staging RDFArchitect at it, smoke-test.
5. **Schedule a window** if you have active users.
6. **Roll out** by changing image tags (Compose / Kubernetes) or replacing the JAR (bare-metal). Restart.
7. **Verify.** Open the welcome page, open a dataset, open the changelog, open SHACL, take a snapshot.

## Version compatibility

| From → To | Notes |
| --------- | ----- |
| 0.x → 1.0 | Run a one-time migration to assign UUIDs to resources that didn't have them in 0.15.x and earlier (RDFA-438). The application does this automatically on first start; back up before the first start of 1.0. |
| 1.x.0 → 1.x.y | Patch upgrade — no schema changes, drop-in. |
| 1.x.y → 1.(x+1).0 | Minor — additive changes only. Drop-in safe. |
| 1.x → 2.x | Major — review breaking changes before proceeding. |

## Schema migration vs. application upgrade

Don't confuse these:

- An **application upgrade** is rolling out a new version of RDFArchitect. The data on disk stays the same.
- A **schema migration** is moving instance data from one CIM version to another (CIM 16 → CIM 17). That uses the [Migration Wizard](/user-guide/migration-wizard) and is independent of RDFArchitect's own version.

You may need both, separately, in the same change window.

## Rollback

If an upgrade fails:

1. Stop the new container / process.
2. Restore the Fuseki backup taken before the upgrade.
3. Start the previous version.
4. File an issue with reproduction details.

A clean rollback is why backups before upgrades are non-negotiable.

## Configuration migration

For minor / patch upgrades, configuration files are forward-compatible — new properties get sensible defaults, removed properties are ignored.

For major upgrades, the changelog lists renamed and removed properties. Update your `application.yml` (and orchestrator manifests / env vars) before starting the new version.

## Frontend / backend skew

Always upgrade frontend and backend to the same version, in either order. The runtime in 1.0 detects mismatches and shows a warning banner; future versions may refuse to operate skewed.

## Browser cache

After an upgrade, end users may need to hard-refresh (Ctrl/Cmd + Shift + R) to pick up the new frontend bundle. The default cache headers force-revalidate `index.html`, so a normal refresh is usually enough. Communicate this in your release announcement.
