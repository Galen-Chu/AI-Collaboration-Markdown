# SYSTEM

> What's shared across all my projects

## Tooling

- Node 20 LTS, TypeScript 5.x, npm
- Postgres 16 everywhere; one database per service
- All times UTC, ISO-8601, no exceptions

## Conventions

- Conventional commits
- A store layer is the only code that touches the DB (pattern, see ARCHITECTURE.md)
- Errors carry a stable machine code; the message is for humans

## Reusable patterns

- TECH.md owns the upgrade policy; bumps get a LOG.md line
- Incidents get an ANALYSIS.md within 48 h
