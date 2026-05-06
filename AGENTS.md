# AGENTS.md

## Cursor Cloud specific instructions

### Codebase overview

This repository is an HTML5 game marketplace called **HGS**. It currently contains 26 static browser games under `comple/` — no backend, no build system, no package manager. All JS libraries (jQuery, CreateJS, Three.js, Construct 2 runtime) are vendored directly in each game directory.

### Running the games

Games **must** be served over HTTP (not `file://`). Construct 2 games explicitly require this. Use any static file server:

```
serve /workspace/comple -l 8080 --no-clipboard
```

or

```
python3 -m http.server 8080 -d /workspace/comple
```

Games are accessed at `http://localhost:8080/<game_name>/<game_name>.htm`.

### Game types

- **Custom JS games** (13): jQuery + CreateJS/Three.js — entry point is `<game_name>.htm`
- **Construct 2 games** (13, suffix `_c2`): use `c2runtime.js` — entry point is `<game_name>.htm`

### Lint / Test / Build

There are currently no lint, test, or build commands — the codebase is purely static assets with vendored dependencies. Future platform code (see `product.md`) will introduce these.

### Important notes

- Node.js must be installed in the environment (used for `serve` static file server via `npx serve` or global install).
- The `product.md` file describes the planned full-stack platform (React/Next.js frontend, Node.js/Python backend, PostgreSQL, Stripe) but none of that is implemented yet.
