# Domain glossary

Terms used across the Excalidraw monorepo product and libraries. Definitions are aligned with code and UX, not external marketing copy.

| Term | Meaning |
|------|---------|
| **Scene** | The drawable content: ordered **elements** plus **app state** (zoom, theme, selected ids, etc.) that the editor persists and restores. |
| **Element** | A shape, text, image, frame, or other object in the scene (`@excalidraw/element` types). Elements have geometry, style, and versioning for sync. |
| **App state** | UI and viewport state (not always the same as “application settings”): selection, open dialogs, scroll, zen mode, collab flags — see `AppState` / `UIAppState` in types. |
| **Excalidraw (component)** | The main React editor exported from `@excalidraw/excalidraw`; renders canvas, toolbars, and editor dialogs. |
| **excalidraw-app** | The full product SPA: wraps `<Excalidraw>` with sharing, collab, Firebase/Plus hooks, analytics, PWA, and shell menus. |
| **Workspace package** | An internal npm package in `packages/*` (`@excalidraw/common`, `math`, `element`, `utils`, `excalidraw`) built and linked via Yarn workspaces. |
| **Collab / collaboration** | Real-time multi-user editing using the app’s collab layer (`socket.io-client`, `Collab.tsx`). Disabled when the app runs inside an **iframe**. |
| **Library (shape library)** | Reusable sets of shapes users can drag onto the canvas; stored locally (IndexedDB via `idb-keyval`) and mergeable from **URL tokens**. |
| **Shareable link** | Flows that encode or load scene data via URL/backend (`ShareableLinkDialog`, collaboration link helpers in `excalidraw-app/data`). |
| **TTD (text to diagram)** | Dialogs and pipeline that turn text (e.g. Mermaid) into canvas elements (`TTDDialog`, `@excalidraw/mermaid-to-excalidraw`). |
| **PWA** | Progressive Web App: service worker registration in `excalidraw-app/index.tsx` for installability and offline-oriented caching. |
| **Excalidraw Plus** | Cloud-oriented export/integration path in the app (`ExportToExcalidrawPlus`, dedicated iframe export route). |
| **Reconcile** | Merging remote and local element lists with version and deletion semantics (`reconcileElements`, `RemoteExcalidrawElement`). |
| **Restore** | Parsing persisted JSON/blob into validated elements and app state (`restoreElements`, `restoreAppState`). |
| **Welcome screen** | First-run hints on an empty canvas (`WelcomeScreen`, `showWelcomeScreen`), layout differs on mobile vs desktop. |
| **Zen mode** | Reduced chrome / focus mode for drawing (editor state flag). |
| **View mode** | Read-only or presentation-style viewing (editor props / state). |
| **Top-level vs iframe** | `getFrame()` returns `"top"` or `"iframe"`; drives collab enablement and some UX constraints. |
| **Conditional exports** | Node resolution entries in package `package.json` that pick `development` vs `production` build artifacts for `@excalidraw/*` packages. |
| **Alias (Vite)** | Dev-time path mapping from `@excalidraw/...` to `packages/.../src` so the app does not require a prior `build:packages`. |

## Related docs

- Product goals and scenarios: `docs/memory/productContext.md`
- Architecture summary: `docs/memory/systemPatterns.md`
- Deep dive: `docs/technical/architecture.md`
