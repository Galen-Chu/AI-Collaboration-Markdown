# OPERATION

> How to run this day to day, in prod

## Environments

| Env | Where | Notes |
|---|---|---|
| prod | atlas.internal:8080 | single instance only — no leader election (DESIGN.md) |

## Deploy

1. `npm run migrate up`
2. Deploy the image
3. Watch the signals below for 10 min

## Monitoring

- Run volume: ~4,000/day expected since the pilot migration (LOG.md 2026-08-01)
- Failed-run rate: page at > 2%
- Caught-up lag after restart: alert if > 60 s

## Rollback

Redeploy the previous image. Migrations are additive-only; no down step needed.

## Environment variables

| Var | Example | Notes |
|---|---|---|
| DATABASE_URL | `postgres://…` | required |
| TICK_MS | 1000 | poll interval |
