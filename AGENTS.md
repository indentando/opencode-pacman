# AGENTS.md!

## What this is

Vanilla JS Pac-Man clone. No build step, no bundler, no package manager.

## Structure

- `src/index.html` — entrypoint; loads scripts via `<script>` tags in order: `maze.js`, `game.js`, `render.js`, `main.js`
- `src/js/` — all game logic (4 files)
- `src/css/style.css` — single stylesheet

## How to run

Open `src/index.html` in a browser. No server required (but ES modules would; this project does not use them).

## Development workflow

Uses **Spec Driven Development**. See the `spec` and `spec-impl` skills in `.agents/skills/` for the workflow. Specs are designed before implementation.

## Conventions

- No modules or imports — files share globals via script load order
- No transpilation — write plain ES5-compatible JS
- Language in UI and comments: Spanish (`lang="es"`)
