# DESIGN

> Why this approach, not another

## Context

Internal tool, single operator, no ops team. Jobs must survive restarts. We already run Postgres for everything else.

## Options

| Option | Pros | Cons |
|---|---|---|
| Poll DB + tick cursor | One dependency (Postgres); simple mental model | Tick latency bounded by poll interval |
| Redis-backed queue | Proven pattern | Second piece of infra to run and back up |
| k8s CronJobs | Native | We don't run k8s for internal tools |

## Decision

Poll-based scheduler over Postgres with a tick cursor. Poll interval (1 s) is far below our finest job granularity (1 min), so latency is a non-issue.

## Consequences

- Correct backfill after downtime becomes our problem → S-12
- No leader election: single instance only, enforced by ops docs (OPERATION.md)
