---
title: Style and Quality Gates
sidebar_position: 11
---

# Style and Quality Gates

The repository ships with a strict-but-uniform set of formatters, linters, and static analyzers. CI enforces every one of them. If something fails locally, run the matching auto-fix command first.

## Backend

| Tool | What it does | How to run | Auto-fix |
| ---- | ------------ | ---------- | -------- |
| Spotless | Format Java + YAML + Markdown. | `mvn -B verify` | `mvn spotless:apply` |
| Checkstyle | Style violations. | `mvn -B -Plint -DskipTests verify` | None — fix manually. |
| SpotBugs | Static bug detection. | `mvn -B -Plint -DskipTests verify` | None — fix manually. |
| license-maven-plugin | Verifies SPDX headers and aggregates third-party licenses. | `mvn -B verify` | `mvn license:format` |

Configuration files live under `backend/config/`:

```
backend/config/
├── checkstyle.xml
├── spotbugs-exclude.xml
└── spotless/
```

### Java code style

Driven by `.editorconfig` and Spotless's google-java-format profile. Key conventions:

- Imports are organised alphabetically with `java.*` first.
- No wildcard imports.
- Lombok is allowed and used widely (`@RequiredArgsConstructor`, `@Builder`, `@Slf4j`).
- MapStruct is used for DTO conversions.
- License header (Apache 2.0 SPDX) is required at the top of every Java file.

### Suppressing a SpotBugs finding

Edit `backend/config/spotbugs-exclude.xml` and add a justified entry. Don't `@SuppressWarnings` at the source level except for unavoidable Lombok/MapStruct interactions.

## Frontend

| Tool | What it does | How to run | Auto-fix |
| ---- | ------------ | ---------- | -------- |
| Prettier | Format Svelte + JS + TS + CSS. | `npm run format` | (formatter) |
| ESLint | Lint JS + TS + Svelte, plus three custom rules. | `npm run lint` | `npm run lint:fix` |
| Vitest | Unit tests. | `npm run test` | — |
| `licenses:generate` | Aggregates third-party licenses. | `npm run licenses:generate` | (regenerator) |

The custom ESLint rules live in `frontend/eslint-rules/`:

| Rule | Enforces |
| ---- | -------- |
| `license-header` | SPDX license header at the top of every source file. |
| `backend-access-via-connection` | No raw `fetch` or hard-coded backend URLs outside `BackendConnection`. |
| `script-structure` | The Svelte `<script>` block must follow imports → `$props` → `$state` → `$derived` → functions → `$effect`. |

### Svelte style

- Prefer `$props`, `$state`, `$derived`, `$effect`.
- Prefer `onclick`, `onkeydown`, etc. (DOM event props).
- **Do not** introduce `export let`, `$:`, `on:click`, or `createEventDispatcher` in new components. Legacy files may still use them; keep changes local.

### Component conventions

- Route-local UI lives under `routes/`.
- Reusable components live under `lib/components/`.
- Dialogs use `bits-ui` modal primitives via the wrappers in `lib/dialog/`.
- Tailwind utilities first, ad-hoc CSS only when utilities don't fit.

## License headers

Every source file in the repository starts with a license header:

```text
// SPDX-License-Identifier: Apache-2.0
// Copyright (c) 2026 SOPTIM AG
```

CI fails if a file is missing the header. The auto-fix tools above add it on save.

## Renovate

Dependency updates are handled by Renovate. Config lives at `.github/renovate.json`. Renovate opens grouped PRs (e.g. "all-non-major") on a schedule. PRs trigger the same CI gate as human PRs and are reviewed in the same way.

## Conventional commits

PR titles must follow [Conventional Commits](https://www.conventionalcommits.org/). The `pr-title.yml` workflow enforces this. Common types:

- `feat:` — new feature.
- `fix:` — bug fix.
- `chore:` — maintenance, deps.
- `docs:` — documentation only.
- `refactor:` — code change without behaviour change.
- `test:` — test-only change.

Append `!` and a `BREAKING CHANGE:` footer for breaking API changes — see [API Stability](./api-stability).

## DCO sign-off

Every commit must carry a `Signed-off-by:` trailer (`git commit -s`). The `dco.yml` bot checks PRs.
