# TEST

> How to verify the system works

## Levels

| Level | Command | Covers |
|---|---|---|
| unit | `npm test` | cron parser, cursor math, config schema |
| integration | `npm run test:int` (docker) | store, locks (S-9), handler registry |
| e2e | `npm run test:e2e` (~2 min) | schedule → tick → run → history (S-3, S-7, S-15) |
| e2e restart | `npm run test:e2e -- --restart` | backfill after downtime (S-12; added from ANALYSIS 2026-07-18) |

## Fixtures

- `test/fixtures/handlers/` — fake handlers, fast and deterministic
- Tests never call real services
