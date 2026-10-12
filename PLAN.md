# Tab Creator – Plan

A public, offline-capable PWA for creating guitar tabs. Output as ASCII, `.tex` and PDF (compiled from LaTeX on the server).

## Decisions

| Area | Decision |
|---|---|
| Audience | Public product, accounts for optional sync |
| Devices | Tablet + desktop, installable PWA, works offline |
| Stack | TypeScript, React, Vite; Node backend; pnpm monorepo |
| Storage | Local-first (IndexedDB); optional account sync, last-write-wins per document |
| Auth | Email magic link |
| Business | Free; PDF compile rate-limited per user/IP |
| Collaboration | Out of scope (read-only share links maybe later) |
| Editing | Visual grid editor + text DSL, both over one document model |
| Notation | Fret numbers, rhythm, techniques, chord names/diagrams, lyrics, sections, repeats, capo, tuning |
| Instruments | 6-string guitar, custom tunings, capo |
| Outputs | ASCII tab, `.tex`, PDF (A4, full song sheet) |
| ASCII rhythm row | Optional toggle, default off |
| PDF renderer | Custom TikZ/LuaTeX, compiled with `lualatex` |
| Hosting | Docker on a VPS, sandboxed TeX container |
| Playback | Not in v1 |

## Architecture

```
packages/core     document schema (Zod), DSL parser/serializer, ASCII renderer/importer, .tex generator
apps/web          React + Vite PWA: SVG editor, DSL pane, ASCII preview, IndexedDB, service worker
apps/server       Node API: magic-link auth, document sync, compile endpoint + job queue
infra/tex         Dockerfile for locked-down lualatex (no shell escape, CPU/mem/time limits, allowlisted packages)
```

- **Document model (versioned JSON)** is the source of truth: metadata, tuning, capo, sections, measures, beats, notes (string, fret, duration, technique flags).
- **ASCII and `.tex` are generated in the browser** from `core`, so they work offline. PDF needs the server.
- **ASCII is lossy by design**: chord diagrams become chord-name lines, rhythm is an optional duration row. Importer and exporter share one grammar; round-trip golden tests cover everything ASCII can represent.
- **Compile service**: never accepts raw LaTeX from users. It takes the document JSON, generates `.tex` server-side with all text escaped, compiles in a sandbox, caches by content hash.

## Phases

1. **LaTeX spike** – hand-write 2–3 representative songs (rhythm, techniques, chord diagrams, lyrics, repeats) in a custom TikZ/LuaTeX style. Validate layout before any UI. Highest risk, goes first.
2. **Core** – schema, DSL, ASCII output/import, `.tex` generator; golden-file tests against spike output.
3. **Desktop editor MVP** – SVG renderer, cursor, keyboard entry, undo/redo, DSL pane, live ASCII preview.
4. **Tablet input** – fret pad, technique palette, touch gestures, responsive layout.
5. **PWA + local-first** – service worker, IndexedDB library, install, offline ASCII/`.tex` export.
6. **Backend** – magic-link auth, sync, compile service, rate limits, content-hash cache.
7. **Polish/launch** – accessibility, onboarding, sample tabs, abuse protection.

## Risks

- TeX layout quality for rhythm + techniques (mitigated by spike).
- Public compile endpoint is a security/DoS surface.
- Fast touch note entry needs early prototyping.
- Notation scope is wide; ship a minimal technique/diagram set first.

## Still open

- Chord diagrams in v1 or v1.1?
- Read-only share links in v1 or later?
- Exact technique set for v1.
