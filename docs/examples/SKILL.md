# SKILL — Add a new job handler

> How to do one recurring task

## When to use

A job needs a type beyond http/shell (e.g. "run this container").

## Inputs

- Handler name and its config schema

## Steps

1. Register the handler in `worker/handlers/index.ts`
2. Extend the config schema in `worker/schema.ts`
3. Add a fixture under `test/fixtures/handlers/`
4. Update SPECIFICATION.md if the behavior is user-visible

## Output

Handler registered, `npm run test:int` green, SPECIFICATION.md updated.
