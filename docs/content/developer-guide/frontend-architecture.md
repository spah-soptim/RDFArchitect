---
title: Frontend Architecture
sidebar_position: 6
---

# Frontend Architecture

The frontend is a SvelteKit 2 application using Svelte 5 runes, Vite 7, Tailwind 4, and Bits UI 2. It is a single-page editor with a small number of routes.

## Stack at a glance

| Concern | Choice |
| ------- | ------ |
| Framework | Svelte 5 (runes) + SvelteKit 2 |
| Bundler | Vite 7 |
| Styling | Tailwind 4 + theme tokens in `app.css` |
| Modal primitives | Bits UI 2 |
| Diagram engine | SvelteFlow + custom layouting |
| Lint | ESLint with three custom rules |
| Test | Vitest |
| State | Svelte runes, custom reactive wrappers, `sharedState.svelte.js` |

## Route map

| Path | Source | Purpose |
| ---- | ------ | ------- |
| `/` | `routes/+page.svelte` | Welcome page, dataset list. |
| `/mainpage` | `routes/mainpage/` | The editor (diagram, navigation, class editor). |
| `/changelog` | `routes/changelog/` | Per-graph history with restore. |
| `/compare` | `routes/compare/` | Two-side comparison view. |
| `/migrate` | `routes/migrate/` | Five-step migration wizard. |
| `/shacl` | `routes/shacl/` | SHACL inspection dialogs (full/class/property). |
| `/prefixes` | `routes/prefixes/` | Per-dataset namespace management. |
| `/layout` | `routes/layout/` | Shared layout primitives, top toolbar. |

`+layout.svelte` at the repository root wires the app shell, top toolbar, and global event bus.

## Reactive wrappers

`src/lib/models/` hosts a small set of "reactive wrappers" that turn the backend's plain JSON shapes into runes-backed Svelte state. The pattern looks like this:

```js
// reactive-class.svelte.js
export class ReactiveClass {
  iri = $state('');
  label = $state('');
  comment = $state('');
  attributes = $state([]);
  associations = $state([]);

  constructor(plain) { Object.assign(this, plain); }

  toPlain() { return { iri: this.iri, label: this.label, /* … */ }; }
}
```

Every domain entity in the model has a wrapper: `ReactiveClass`, `ReactiveAttribute`, `ReactiveAssociation`, `ReactiveNamespace`, `ReactivePackage`, `ReactiveShape`, …

Why wrappers? Because runes-based reactivity needs `$state` declarations on the *fields* you mutate, and SvelteKit serialises plain objects on transport. Wrappers let us hydrate plain JSON into reactive instances cleanly.

## Backend communication

`BackendConnection` (in `src/lib/api/`) is the only place that constructs URLs to the backend. Components import it and call methods like:

```js
import { backend } from '$lib/api/backend.js';

const classes = await backend.classes.list({ datasetId, graphId });
const updated = await backend.classes.edit({ datasetId, graphId, classIri, body });
```

Three rules:

1. **Always go through `BackendConnection`.** Don't build URLs by hand.
2. **Preserve `credentials: "include"`.** Required for the session cookie to be sent.
3. **Use the runtime config.** `PUBLIC_BACKEND_URL` is injected at container start; never hardcode `http://localhost:8080`.

The method names mirror the REST hierarchy. Adding a new endpoint means adding the matching method in `backend.js`.

## State and data flow

There are three layers of state:

| Layer | Where | Lifetime |
| ----- | ----- | -------- |
| **Component-local** | Inside `+page.svelte` or `Component.svelte` | Component lifetime. |
| **Route-shared** | Reactive instances passed via context. | Route lifetime. |
| **Cross-route** | `sharedState.svelte.js` (active dataset, active graph, undo stack hooks). | Browser session. |

There is no Redux/Pinia/Zustand-style global store. Don't add one. Reuse the existing primitives:

- `StateValuePair` — a pair of (current value, original value) for unsaved-change tracking.
- `sharedState.svelte.js` — singleton with the active dataset/graph and global event hooks.
- The event bus in `eventhandling/` — fires reload triggers when one route mutates state another route is showing.

## Validity rules

Per-component validation logic lives next to the component (e.g. `classEditor/validity.js`). The pattern is a pure function that takes the current state and returns either `null` or an array of `{ field, message }` objects. The component renders the messages inline and disables save while there are errors.

## Script-block ordering convention

Svelte components in this repo follow a strict ordering of `<script>` content, documented in `frontend/docs/script-structure.md`:

1. Imports.
2. `$props` declaration.
3. Local `$state`.
4. `$derived`.
5. Functions.
6. `$effect`.

ESLint enforces this via a custom rule. Don't fight it — the linearity makes diffs much easier to read.

## Custom ESLint rules

Three custom rules live in `frontend/eslint-rules/`:

- **License header** — every source file must start with the SPDX license header.
- **Backend access guard** — flags any `fetch(`, `axios.`, or hard-coded backend URLs outside `BackendConnection`.
- **Script structure** — enforces the order above.

## Styling

Tailwind 4 with theme tokens defined in `src/app.css`. Components reach for utility classes; design tokens are referenced via CSS variables (`var(--ifm-color-primary)` etc.). Don't add ad-hoc colors — extend the token list.

## Diagram rendering

`src/lib/rendering/` holds:

- A SvelteFlow-based renderer for class boxes and edges.
- A custom layouter that assigns initial positions for a freshly opened package.
- Helpers that translate model objects into SvelteFlow nodes and edges.
- Mermaid-based fallbacks for printable views.

When a model changes, the rendering layer subscribes to the relevant reactive wrappers and recomputes only the affected parts.

## Internationalisation

There is no i18n infrastructure in 1.0 — UI strings are hardcoded English. If you add a feature, keep strings in template literals and don't extract them to a translation table; there is no current consumer for that table and adding one without a translation pipeline would be wasted churn.
