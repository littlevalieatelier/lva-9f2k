# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file static web app (`index.html`) for Little Valie Atelier to track Instagram/social reels through production stages (Idea → Scripted → Filming → Editing → Posted). No build step, no dependencies, no framework — plain HTML/CSS/JS in one file.

## Development

There is no build, lint, or test tooling in this repo. To work on it, open `index.html` directly in a browser (or serve the directory with any static file server) and reload after edits.

## Architecture

Everything — markup, styles, and logic — lives in `index.html` as a single page:

- **Access gate**: a client-side password check (`PASSWORD` constant, `tryUnlock()`) gates the `#app` view behind a `#gate` overlay. The password is a plaintext string in the JS source and unlock state is just a `sessionStorage` flag (`lva_unlocked`) — this is a soft deterrent for a small private tool, not real auth.
- **Data model**: reels are plain objects `{ id, code, title, hook, stage, createdAt }` held in the in-memory `reels` array and persisted as JSON to `localStorage` under `lva_reels`. There is no backend — all data is local to the browser/device. `code` (e.g. `LVA-001`) is generated from a separate persisted counter (`lva_reel_counter` / `nextNum`).
- **Render loop**: state-mutating functions (`saveReel`, `setStage`, `deleteReel`) mutate the `reels` array, call `persist()` to write to `localStorage`, then call `render()` to fully re-render `#stats` and `#list` from scratch via `innerHTML` — there is no diffing or component structure.
- **Stages**: the `STAGES` array (`Idea, Scripted, Filming, Editing, Posted`) is the single source of truth for stage labels/order; a reel's `stage` field is just an index into it. Stage button styling keys off `.s0`–`.s4` classes matching that index.
- User-provided text (title, hook) is escaped via `escapeHtml()` before being interpolated into `innerHTML` — preserve this when touching the render path.
