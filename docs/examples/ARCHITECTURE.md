# ARCHITECTURE

> What's in the stack

## Components

```
api/        HTTP endpoints — schedules, run history
scheduler/  tick loop — decides what is due
worker/     executes jobs, records results
store/      the only layer that touches Postgres
```

## Data flow

scheduler polls store for due ticks → worker runs the job → result (exit code, stderr, duration) written back through store → api reads it.

## Boundaries

- store/ owns all SQL; no other layer imports `pg`
- worker knows job types, not schedules; scheduler knows schedules, not job types
