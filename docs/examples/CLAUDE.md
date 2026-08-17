# CLAUDE

## Rules for AI agents

- Never edit `worker/handlers/index.ts` without also updating SPECIFICATION.md.
- Tests live in `test/`, fixtures in `test/fixtures/`. Use existing patterns.
- The store layer (`store/`) is the only code that touches Postgres.
- Before adding a dependency, check TECH.md for the upgrade policy.

## Commands

- `npm test` — unit tests
- `npm run test:int` — integration (needs docker)
- `npm run test:e2e` — full loop (~2 min)
