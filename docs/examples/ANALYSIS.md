# ANALYSIS — 2026-07-18 restart incident

> What did we learn, after the fact

## What happened

Deploy restart, 30 min downtime. 6 ticks missed; 1 replayed; 5 lost silently. A pilot team noticed, not us.

## Interpretation

Backfill advanced the cursor only after a tick completed, so one slow first run exited the replay loop. The loss was silent because nothing anywhere asserted a replay count. The gap was between "backfill exists" and "backfill is correct" — we shipped the former and assumed the latter.

## Actions

| Action | Owner | Lands in |
|---|---|---|
| Spec replay correctness | j.lee | S-12 |
| Acceptance criterion for replay | j.lee | C-003 |
| TROUBLESHOOTING entry | j.lee | T-4 |
| E2E: restart with missed ticks | t.chen | TEST.md |
