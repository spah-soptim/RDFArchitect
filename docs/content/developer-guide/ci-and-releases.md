---
title: CI and Releases
sidebar_position: 13
---

# CI and Releases

What runs when, and how a release happens.

## Workflows

| Workflow | Triggers | Purpose |
| -------- | -------- | ------- |
| `backend-ci.yml` | PR / push to main | `mvn verify`, license check. |
| `frontend-ci.yml` | PR / push to main | lint / test / build / license check. |
| `deploy-docs.yml` | PR / push to main / manual | Builds Docusaurus site, deploys to GitHub Pages on main. |
| `pr-title.yml` | PR | Conventional Commits enforcement. |
| `publish-test-images.yml` | tag push | Publishes versioned Docker images to GHCR. |

## CI matrix

Backend and frontend workflows run on `ubuntu-latest` with the toolchain versions pinned to match the repo:

- Backend: Eclipse Temurin 25, Maven 3.9.x.
- Frontend: Node 24, npm 11.

The docs workflow uses Node 20 (sufficient for Docusaurus).

## Status badges

The README pins:

- Backend CI status.
- Frontend CI status.
- Latest tag.
- License.

Add a docs CI badge here if/when you start gating PRs on the docs build. The current `deploy-docs.yml` builds on every PR but only deploys from main.

## Releases

Releases are tag-driven and follow semantic versioning. Cutting a release:

1. Update `CHANGELOG.md`: move entries from `[Unreleased]` into a new versioned section with the release date.
2. Bump versions if anything is hard-coded outside Maven/npm metadata.
3. Open a release PR (`chore(release): vX.Y.Z`).
4. Merge.
5. Tag main: `git tag vX.Y.Z && git push --tags`.
6. The `publish-test-images.yml` workflow builds and pushes Docker images to GHCR with both `vX.Y.Z` and `latest` tags.
7. Create a GitHub Release pointing at the tag, copying the changelog section.

The release notes are derived from `CHANGELOG.md`, not from auto-generated commit messages.

## Docker images

Published to:

- `ghcr.io/soptim/rdfarchitect-backend:vX.Y.Z`
- `ghcr.io/soptim/rdfarchitect-frontend:vX.Y.Z`

The images include the `APP_VERSION` build-arg, surfaced in `/api/version` (backend) and the about dialog (frontend).

## Hotfixes

For urgent fixes off a release tag:

1. Branch from the tag: `git checkout -b hotfix/vX.Y.Z+1 vX.Y.Z`.
2. Apply the fix, update the changelog (use a `[X.Y.Z+1]` section).
3. Open a PR against `main` first so the fix lands on the trunk.
4. Tag the hotfix branch and let `publish-test-images.yml` build the images.

Do not push tags from a long-lived hotfix branch without merging back to main; the trunk should always be ahead.
