# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

"Water today" — a single-file water-intake tracker PWA. No build step, no dependencies,
no package manager, no tests, no linter. `index.html` is the entire application
(HTML + CSS + JS inline). Edits are made directly to that file and are live on reload.

Not a git repository (yet). The intended deployment is GitHub Pages from a repo named
`water-tracker`, served at a subpath — which is why `manifest.json` and `sw.js` use
relative `./` paths. Keep all paths relative.

## Running it

Opening `index.html` via `file://` works for the UI, but the service worker and the
install prompt require an http(s) origin. To exercise the PWA behaviour:

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000. Service workers are only allowed on localhost or https.

## Architecture

- **`index.html`** — the whole app. Three concerns live in it:
  - `:root` CSS custom properties hold the palette (`--ice`, `--lagoon`, `--shallow`,
    `--deep`, `--mist`, `--foam`, `--muted`). Change colours there, not inline.
  - The glass is an inline SVG with a `clipPath` (`#cup`) and a `<g id="water">` holding
    two animated wave paths. Fill level is driven purely by `transform: translateY(...)`
    on `#water`, computed in `render()` from the cup geometry (top `y=14`, bottom `y=252`).
    If you change the cup path, update those two constants in `render()` to match.
  - All logic is in the single trailing `<script>`: `load`/`save`/`rollover`/`add`/`render`,
    plus delegated click handlers. `render()` is the only place that touches the DOM —
    mutate `state`, call `save()`, then `render()`.
- **`sw.js`** — cache-first-on-failure service worker (network first, falls back to cache).
- **`manifest.json`** + `icon-192.png` / `icon-512.png` — installability on Android.
- **`WaterToday.exe`** — prebuilt, unsigned Neutralinojs Windows build. No source for it is
  in the repo; it is not rebuilt from these files by anything here.

### State

`localStorage` key `water-log-v1`, shape `{ day: "YYYY-MM-DD", entries: [{ ml, t }] }`.
`day` comes from `toLocaleDateString('en-CA')` (chosen because it yields ISO-like dates).
Entries are **newest-first** (`unshift`), and the log's Remove buttons pass the array index,
so any change to insertion order must be mirrored in the remove handler. Day rollover is
checked on load, on every `add()`, and by a 60-second `setInterval`.

### Things that must be kept in sync when changing files

- Bumping any cached asset → bump the cache name `C='water-v1'` in `sw.js`, or clients keep
  the stale copy. Adding a new file → add it to the `addAll([...])` list too.
- Icon filenames appear in `manifest.json`, `sw.js`, and `index.html` (`apple-touch-icon`).
- Theme colour `#F3F8FC` appears in `manifest.json` (`background_color`, `theme_color`) and
  in the `<meta name="theme-color">` in `index.html`.
- The daily goal is `const GOAL = 800`, but "800 ml" is also hardcoded in the user-facing
  copy (the `.tally` paragraph, the initial `#big`/`#sub` markup, and the "over your 800 ml
  limit" string in `render()`). Change all of them together.

## Conventions

- No frameworks, no bundler, no npm — keep it dependency-free. The only external resource
  is the Google Fonts stylesheet (Bricolage Grotesque for numbers/headings, Figtree for
  body); it is deliberately not in the service worker cache.
- Event handling uses delegation on container elements (`#chips`, `#logWrap`) rather than
  per-element listeners, since the lists are re-rendered as innerHTML.
- `localStorage` access is wrapped in try/catch so a blocked-storage browser still runs.
- Accessibility and motion preferences are already handled (`aria-live` readout,
  `:focus-visible` rings, `aria-label`s on Remove, `prefers-reduced-motion` disabling the
  wave animation and fill transition). Preserve these when editing markup or CSS.
