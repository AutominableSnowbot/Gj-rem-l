# Gjøremål

A simple, private, offline-friendly to-do app. Norwegian for "tasks" or "things to do."

Same architecture as [Spiskammer](https://github.com/AutominableSnowbot/pantry) — built as a Progressive Web App with all data stored locally in your browser.

## Features

- **Multiple lists** — Main (always present) plus any extras you create (Work, Home, Errands)
- **Tap to complete** — circle checkbox on each task
- **Optional due dates** — long-press a task to set or clear a date. Overdue items show in red, today in amber, soon in yellow
- **Voice input** — speak tasks naturally (Android Chrome only; the button is hidden where unsupported)
- **AI helper** — copy a prompt, paste it into any AI chat with a photo, screenshot, or text. The AI returns JSON of tasks you can paste back in
- **Paste / import JSON** — bring tasks in from external sources
- **Show/hide completed** — keep a record of what you've done, or hide it
- **Offline** — works fully offline after first load. Data stays on your phone
- **Long-press menu** — edit text, set due date, delete

## Install

Deploy to GitHub Pages, then on your phone:

- **Android**: open in Firefox or Chrome → browser menu → Add to Home Screen
- **iOS**: open in Safari → Share button → Add to Home Screen

## File structure

- `index.html` — the entire app (React + Tailwind via CDN, single file)
- `manifest.json` — PWA manifest
- `sw.js` — service worker for offline support
- `icon-192.png`, `icon-512.png` — app icons

No build step. Edit the HTML, commit, GitHub Pages serves it.

## Storage keys

Uses these localStorage keys:

- `gjoremal-lists` — array of lists with their tasks
- `gjoremal-active-list` — id of currently active list
- `gjoremal-show-completed` — whether completed tasks are visible

## Updating

When you change `index.html`, bump the cache version in `sw.js` (e.g. `gjoremal-v1` → `gjoremal-v2`) so the service worker fetches the new version.
