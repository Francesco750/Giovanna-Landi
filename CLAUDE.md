# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A PWA workout tracker for client Giovanna Landi. Delivered as a single self-contained HTML file with no build tools or package managers. Hosted on GitHub Pages at `https://francesco750.github.io/Giovanna-Landi/`.

## Running the App

Open `giovanna_workout.html` directly in a browser (`file://`), or via GitHub Pages. The PWA installs via `manifest.json` + `sw.js` (cache-first service worker under key `gl-workout-v1`).

`index.html` exists solely as a redirect to `giovanna_workout.html`.

## Architecture

Everything lives in `giovanna_workout.html` with:

- **Data layer** (`SESSIONS` array): All workout content declared as a JS data structure. Two sessions (A, B), each with an `activation` string (shown in the warmup box) and muscle `groups`, each containing `exercises`. Each exercise has a `weeks[]` array of 4 entries (one per week, alternating pattern: W1=W3, W2=W4).

- **Week entry format**: `{sets, reps, mod?, rec}` where `mod` is optional (`'isomix'`, `'iso'`, `'drop'`) and `rec` is recovery in seconds. Cardio entries use `{type:'cardio', dur}`. Exercises with no rest use `rec: 0`.

- **Prescription types**:
  - `mod:'isomix'` → every 5 reps hold isometric 5 sec — rendered with purple badge
  - `mod:'drop'` + `reps:'8+8+8'` → drop weight every 8 reps — rendered with red badge
  - `mod:'iso'` → hold isometric 10 sec at end of set — rendered with cyan badge
  - no mod → standard sets × reps

- **Rendering** (`buildAll` / `buildSession`): On init, `buildAll()` generates all session HTML from `SESSIONS` and injects it into `#s0`, `#s1`. The current week is read from `localStorage` key `gl_week`.

- **Week switching** (`switchWeek`): Does NOT re-render — surgically updates prescription badges (`#pres-{id}`, `#cpres-{id}`), timer button wrappers (`#twrap-{id}`), and set-tracker dots in the existing DOM.

- **Set tracker dots**: Built per-exercise by `buildDots()` / `initTrackers()`. Purely in-memory — reset on page reload, intentionally not persisted.

- **Weight persistence**: Each exercise kg input auto-saves to `localStorage` under key `gl_kg_{exercise.id}`. Exercise IDs are `a01`–`a11`, `b01`–`b11`, plus cardio IDs `a-c`, `b-c`.

- **Recovery timer**: Fixed bottom bar (`#tmr-bar`). Recovery time is per-week (from `w.rec`), updated on week switch via `#twrap-{id}`. Uses `setInterval` + SVG stroke-dashoffset for the ring. Plays Web Audio API beeps at 1–3 sec remaining; vibrates on completion.

- **Wake Lock**: Toggleable via the "Schermo ON" pill; re-requests on `visibilitychange`.

- **Theme**: Dark background `#0c0c0c`, fuchsia accent `#d946ef`. Fonts: Bebas Neue (headings) + DM Sans (body) via Google Fonts CDN.

## Key Conventions

- Exercise `id` fields (`a01`, `b03`, etc.) must be unique — used as localStorage keys and DOM `data-eid` selectors.
- To add a new week: extend every `weeks[]` array and add a `<button class="wk-btn">` in the week bar HTML.
- To add a new exercise: add an entry to the appropriate group's `exercises[]` array; the renderer handles it automatically.
- `rec: 0` means no timer button is shown for that exercise.
