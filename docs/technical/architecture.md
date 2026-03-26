# Architecture (detailed)

This document expands on `docs/memory/systemPatterns.md` with structure, flows, deployment, and non-obvious behaviors inferred from the codebase.

## 1. Repository topology

```text
excalidraw-monorepo/
├── excalidraw-app/          # Product SPA (Vite + React)
├── packages/
│   ├── common/              # Shared constants, utils, env helpers
│   ├── math/                # Geometry and math types
│   ├── element/             # Element model, mutations
│   ├── utils/               # General helpers
│   └── excalidraw/          # Editor UI, public React API
├── examples/                # Embed integrations (Next.js, script tag)
├── firebase-project/        # Firebase-related config
├── scripts/                 # buildPackage, locales, release, etc.
└── .github/workflows/       # CI
```

## 2. Runtime composition of the product app

1. **Bootstrap** — `excalidraw-app/index.tsx` mounts React, registers the PWA service worker, renders the shell.
2. **Error boundary** — `TopErrorBoundary` wraps the tree in `App.tsx`.
3. **Global state** — Jotai `Provider` with `appJotaiStore` (`app-jotai.ts`) for shell concerns (collab handles, quotas, flags).
4. **Imperative API bridge** — `ExcalidrawAPIProvider` (from `packages/excalidraw`) lets the shell call `useExcalidrawAPI()` outside the inner `<Excalidraw>` subtree.
5. **Editor** — `<Excalidraw>` from `@excalidraw/excalidraw` hosts canvas, tools, dialogs; internal state uses the package’s Jotai layer (`editor-jotai`).

## 3. Build and consumption modes

| Mode | How `@excalidraw/*` resolves | Typical use |
|------|------------------------------|-------------|
| App dev | Vite `resolve.alias` → package **source** (`*.ts` / `*.tsx`) | Fast iteration in monorepo |
| Published npm | `package.json` **conditional exports** (`development` / `production`) → `dist/*` | External apps and examples after `build:packages` |
| Docker | `yarn build:app:docker` → static assets served by nginx | Production container |

Library bundles are produced by `scripts/buildPackage.js` (esbuild + Sass), not by the app’s Vite build.

## 4. Major data and integration flows

- **Scene** — Elements and app state are serialized/restored via package APIs (`restore`, `reconcile`, blob import/export).
- **Collaboration** — `excalidraw-app/collab/` plus `socket.io-client`; sharing UI in `share/`. Gated when running inside an iframe (see implicit behaviors below).
- **Library** — Shape libraries persisted with `idb-keyval`; URL parameters can seed or merge libraries (`parseLibraryTokensFromUrl` + `useHandleLibrary` in `App.tsx`).
- **Backend / cloud** — Optional Firebase, export-to-backend helpers under `excalidraw-app/data/`, Sentry in `excalidraw-app/sentry`.

## 5. Frontend build details (Vite)

- **Output** — `excalidraw-app/build`.
- **Env** — `loadEnv` / `envDir` point at the **repository root** (`../` relative to the Vite config), so `.env` files next to root `package.json` apply to the app dev server and builds.
- **Chunking** — `manualChunks` separates locale JSON (except English and percentages), and heavy deps (e.g. Mermaid, CodeMirror/Lezer) for caching and load performance.
- **Fonts** — `.woff2` assets get stable paths under `fonts/<family>/`.

## 6. Quality gates

- **Vitest** mirrors Vite aliases (`vitest.config.mts`), **jsdom** environment, `setupTests.ts` includes canvas mocking.
- **Coverage** — Thresholds and `ignoreEmptyLines: false` are intentional (see implicit behaviors).
- **Static checks** — `tsc`, ESLint (`@excalidraw/eslint-config`), Prettier at repo root.

## 7. Implicit runtime and build behaviors (easy to miss)

These are not always visible in user-facing docs; they matter for embeds, CI, and local setup.

1. **Collaboration disabled in iframes** — `isRunningInIframe()` in `@excalidraw/common` uses `window.self === window.top`. If that check throws (e.g. cross-origin parent), the code treats the context as **iframe** and collaboration stays off (`excalidraw-app/App.tsx` → `isCollabDisabled`).
2. **App env files live at monorepo root** — Because `envDir` is `../`, developers who only look under `excalidraw-app/` may miss that **root** `.env*` files drive `VITE_*` variables (including `VITE_APP_PORT`).
3. **Locale split vs first-load/offline** — `manualChunks` excludes `en.json` and `percentages.json` from the “per-locale” chunks so they ship with the core bundle: first paint and PWA precache stay aligned with the previous CRA-style behavior (comment in `excalidraw-app/vite.config.mts`).
4. **Vitest hook ordering** — `sequence.hooks: "parallel"` is set explicitly because Vitest v2’s serial hook stack breaks monorepo test setup expectations (comment in `vitest.config.mts`).
5. **Coverage empty lines** — `coverage.ignoreEmptyLines: false` changes line counts versus Vitest defaults; thresholds were tuned for that (same config file).
6. **Library from URL** — On load, the shell calls `parseLibraryTokensFromUrl()` so query parameters can add library items without a separate import step (`excalidraw-app/App.tsx`).

## 8. Sources in code

- `excalidraw-app/App.tsx`, `excalidraw-app/index.tsx`, `excalidraw-app/vite.config.mts`, `excalidraw-app/app-jotai.ts`
- `packages/excalidraw/index.tsx`, `packages/excalidraw/data/library.ts`
- `packages/common/src/utils.ts` — `getFrame`, `isRunningInIframe`
- `scripts/buildPackage.js`, root and package `package.json`
- `vitest.config.mts`, `Dockerfile`, `docker-compose.yml`
