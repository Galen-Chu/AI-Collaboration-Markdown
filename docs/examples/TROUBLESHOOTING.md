# TROUBLESHOOTING

## T-4: Backfill replays only one missed tick after restart

**Symptom:** Atlas restarted after 30 min downtime; expected 6 backfilled runs, got 1.

**Cause:** Backfill loop queries next tick but doesn't advance cursor until the tick completes. If the first backfilled run is slow, the loop exits before the remaining ticks.

**Fix:** Advance cursor before executing each tick, not after (PR pending).

**Escalation:** If backfill still misbehaves after fix, page j.lee.
