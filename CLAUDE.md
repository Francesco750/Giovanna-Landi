# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A PWA workout tracker for client Raffaele Pannullo, built by trainer Giovanna Landi. Delivered as a single self-contained HTML file with no build tools or package managers.

## Running the App

Open `raffaele_workout.html` directly in a browser (`file://`), or serve via any static file server. The PWA installs via `manifest.json` + `sw.js` (cache-first service worker under key `rp-workout-v1`).

`index.html` exists solely as a redirect to `raffaele_workout.html`.

## Architecture

Everything lives in `raffaele_workout.html` (~604 lines):

- **Data layer** (`SESSIONS` array, lines ~191–318): All workout content is declared as a JS data structure. Three sessions (A, B, C), each with muscle groups and exercises. Each exercise has a `weeks[]` array of 5 entries (one per week), where each entry is `null` (week skipped), `{type:'test', reps}`, `{type:'cardio', dur}`, or `{sets, reps, rir}`.

- **Rendering** (`buildAll` / `buildSession`): On init, `buildAll()` generates all session HTML from `SESSIONS` and injects it into `#s0`, `#s1`, `#s2`. The current week (`currentWeek`) is read from `localStorage` key `rp_week`.

- **Week switching** (`switchWeek`): Does NOT re-render — it walks the `SESSIONS` data and surgically updates prescription badges (`#pres-{id}`, `#cpres-{id}`) and set-tracker dots in the existing DOM.

- **Set tracker dots**: Built per-exercise by `buildDots()` / `initTrackers()`. Purely in-memory — reset on page reload, intentionally not persisted.

- **Weight persistence**: Each exercise kg input auto-saves to `localStorage` under key `rp_kg_{exercise.id}`. Exercise IDs are `a01`–`a06`, `b01`–`b06`, `c01`–`c06`, plus cardio IDs `a-c`, `b-c`, `c-c`.

- **Recovery timer**: Fixed bottom bar (`#tmr-bar`). Uses `setInterval` + SVG stroke-dashoffset for the ring animation. Plays Web Audio API beeps at 1, 2, 3 seconds remaining. Vibrates on completion via `navigator.vibrate`.

- **Wake Lock**: Toggleable via the "Schermo ON" pill; re-requests on `visibilitychange`.

- **Fonts**: Bebas Neue (headings) + DM Sans (body) via Google Fonts CDN. No other external dependencies.

## Key Conventions

- Exercise `id` fields (`a01`, `b03`, etc.) must be unique — they are used as localStorage keys and DOM `data-eid` selectors.
- To add a new week: extend every `weeks[]` array to length 6 and add a new `<button class="wk-btn">` in the week bar HTML.
- To add a new exercise: add an entry to the appropriate group's `exercises[]` array; the renderer handles the rest automatically.
- Jump Sets (`isJs: true`) are paired exercises — the group must have `isJs: true` and each exercise in the pair must have `isJs: true`. They render with a gold left border and a `JS` badge.
- `null` in `weeks[]` means the exercise is not prescribed that week (renders as `—`). Week 5 (index 4) is `null` for all strength exercises by design.
