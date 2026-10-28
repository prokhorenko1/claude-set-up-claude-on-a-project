# Notes

## CLAUDE.md: what's in it and what's not

**In it:** a one-line description, the commands I actually run (`dev`, `test`, a single test file, a test-name filter, `lint`) and the CI gate, a short architecture section (`server.js` exports `app` without listening, one router per resource in `routes/`, `db/store.js` as the only data layer with shared in-memory state), and conventions Claude can't safely guess: CommonJS only, routes go through the store, the JSON error shape and status codes, `Number(req.params.id)` because the store uses `===`, `node:test` + `supertest` instead of Jest, and no exact list-length assertions because tests share store state.

**Left out:** file-by-file listings and anything Claude can see by reading the code, setup and onboarding steps that are already in the README, one-off tasks, and anything from `.env`. Every line costs context in every session, so I kept only the rules that stop Claude from making a plausible but wrong choice.

## Permission rules

- **allow** `Bash(npm test:*)`: the tests are safe and run often, so there's no point being asked every time.
- **ask** `Bash(git push:*)`: pushing is outward-facing, so I want to confirm each one.
- **deny** `Read(./.env)` and `Bash(git push --force:*)`.

Without the deny rules, Claude could read real secrets from `.env` into the conversation, where they might get echoed into output, commits or logs. It could also force-push and overwrite remote history that other people depend on.

## Verification

In a fresh session, `/memory` lists the project `CLAUDE.md` as loaded, and `/permissions` shows the allow, ask and deny rules above. Asking "How do I run the tests here?" gets an answer from `CLAUDE.md` without any extra explanation.
