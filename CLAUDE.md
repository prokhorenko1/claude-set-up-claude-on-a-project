# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

A small Express REST API (users + health check) backed by an in-memory store, used as the starter project for a Claude Code course.

## Commands

- `npm run dev` — start the API with `node --watch` on http://localhost:3000 (`PORT` env overrides)
- `npm test` — run all tests with the built-in Node test runner (`node --test`)
- `node --test tests/users.test.js` — run one test file
- `node --test --test-name-pattern="404"` — run tests whose name matches a pattern
- `npm run lint` — ESLint over the whole repo

CI (`.github/workflows/ci.yml`) runs `npm run lint` then `npm test` on Node 22; both must pass.

## Architecture

- `server.js` builds the Express app, mounts one router per resource (`/users`, `/health`), and exports `app`. It only calls `listen()` when run directly (`require.main === module`), so tests import `app` without opening a port.
- `routes/` holds one `express.Router()` file per resource. A new resource means a new file here plus an `app.use(...)` line in `server.js`.
- `db/store.js` is the only data-access layer: plain functions over a module-level array. State is shared across every request and every test in a process, and resets on restart.

## Conventions

- CommonJS only: `require` / `module.exports`, not `import` / `export` (ESLint is configured with `sourceType: "script"`).
- Route handlers never touch data directly; add a function to `db/store.js` and call it from the route.
- Errors are JSON `{ "error": "<message>" }` with the matching status (400 for invalid input, 404 for missing records); successful creates return 201 with the created object.
- Route params are strings; convert with `Number(req.params.id)` before passing to the store (the store compares with `===`).
- Tests use `node:test` + `node:assert` + `supertest` against the exported `app`, not Jest/Mocha and not a live server. Tests that create data mutate shared store state, so don't assert exact list lengths.
- Style: double quotes, semicolons, a short `// METHOD /path — what it does` comment above each route handler.
