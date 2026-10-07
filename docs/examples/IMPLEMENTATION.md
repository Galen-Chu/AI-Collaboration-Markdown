# IMPLEMENTATION

> How it was actually built

## As-built

- Tick cursor lives in `store.state` (`key = 'tick_cursor'`). Today it advances only after a tick completes — the fix to advance before executing is PR-pending (T-4).
- Backfill is the same loop, started from the last recorded tick instead of now.

## Deviations from ARCHITECTURE

| Intended | Actual | Why |
|---|---|---|
| Handlers auto-discovered, one file per handler | Registry in `worker/handlers/index.ts` | Discovery broke test fixtures; explicit registry wins |

## Gotchas

- pg pool max must be ≥ 5 — backfill bursts open parallel run-record writes
- pg 8.11 changed notice propagation; two integration tests assert on notices (see TEST.md)
