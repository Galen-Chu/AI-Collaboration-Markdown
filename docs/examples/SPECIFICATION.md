# SPECIFICATION

> What it does, precisely

## Functional

| ID | Rule | Priority |
|---|---|---|
| S-1 | A schedule runs an HTTP request or a shell command | Must |
| S-2 | Schedule times are UTC | Must |
| S-3 | `atlas schedule create` validates cron syntax at creation time | Must |
| S-7 | Failed runs are recorded with exit code and stderr | Must |
| S-9 | A schedule never has two concurrent runs | Must |
| S-12 | Backfill correctly replays missed ticks after restart | Must |
| S-15 | Run history API returns last 50 runs by default | Should |
| S-16 | Run records are retained 90 days | Should |

## Non-functional

- Tick loop latency < 1 s at 5,000 runs/day
- Restart-to-caught-up < 60 s after 30 min downtime

## Open questions

- Should backfill replay missed ticks sequentially or in parallel? (blocked on T-4 fix)
