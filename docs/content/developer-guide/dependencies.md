---
title: Dependencies
sidebar_position: 12
---

# Dependencies

How dependency updates are handled and what to do when you change one.

## Renovate

[Renovate](https://docs.renovatebot.com/) opens dependency-update PRs automatically. Config lives at `.github/renovate.json`. The default schedule is weekly with grouped non-major updates.

PRs from Renovate go through the same CI gate as human PRs. Review them like any other PR — read the changelog, check breaking changes, run smoke tests if the dependency is critical (Spring, Jena, Svelte, Vite).

## When you change a backend dependency

1. Update `backend/pom.xml`.
2. Run:

   ```bash
   cd backend
   mvn -B verify
   mvn -B org.codehaus.mojo:license-maven-plugin:add-third-party
   ```

3. Commit the updated `backend/LICENSES-THIRD-PARTY.md`.

CI verifies the file is in sync. A stale license file fails the build.

## When you change a frontend dependency

1. Update `frontend/package.json`.
2. Run:

   ```bash
   cd frontend
   npm install
   npm run licenses:generate
   ```

3. Commit the updated `frontend/LICENSES-THIRD-PARTY.md` and `frontend/package-lock.json`.

CI verifies the file is in sync.

## Compatibility constraints

| Component | Constraint |
| --------- | ---------- |
| Java | 25 — pinned by Spotless, Checkstyle, and SpotBugs configs. |
| Spring Boot | 4.x — major version upgrades are deliberate, separate PRs. |
| Jena | 5.x — major version upgrades require regression-testing the SHACL generator and migration composer. |
| Svelte | 5.x with runes — do not adopt legacy syntax in new code. |
| Tailwind | 4.x — config is in `app.css`, not a separate config file. |
| SvelteKit | 2.x. |
| Vite | 7.x. |

## Adding a new dependency

Before adding, ask:

- Can the existing toolchain do this already? (Tailwind, bits-ui, Jena, Spring all cover a lot.)
- Is the licence Apache-2.0-compatible? (See `LICENSES-THIRD-PARTY.md` for what's already accepted.)
- Is it actively maintained? (Last commit, open issues, security advisories.)

Then:

1. Add it to `pom.xml` or `package.json`.
2. Use it.
3. Regenerate the license file (see above).
4. Mention the new dependency in the PR description.

## Removing a dependency

If you remove the last consumer of a transitive dependency, regenerate license files in the same commit. Otherwise CI fails on the next unrelated change.
