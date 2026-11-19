# Scoring, streaks, badges

Computed only by `progress-service` after server-side verification. Clients never send a score.

## Level score

```
score = base(difficulty)
      × speedFactor
      × (1 - hintPenalty)
      + qualityBonus
      + noRegressionBonus
```

| Part | Rule |
|---|---|
| `base` | 100 × difficulty (1-5) |
| `speedFactor` | 1.0 up to the estimated time, linear decay to 0.6 at 3× estimate. Never below 0.6 |
| `hintPenalty` | Tier 1: 5%, Tier 2: 15%, Tier 3: 30% (cumulative, capped at 50%) |
| `qualityBonus` | 0-20 from Ollama rubric (root cause, readability). **Advisory; capped so it can't flip pass/fail** |
| `noRegressionBonus` | +10 if all regression-trap tests pass first try |
| Failed attempts | -2% per failed server verification after the 3rd, floor at 70% of base |

Pass/fail = hidden tests only.

## Streaks
- Streak day = at least one verified level completion (or daily challenge) in the player's local day.
- Store the player's timezone; compute on the server.
- **Freeze:** 1 freeze earned per 7-day streak, max 2 banked. Auto-consumed on a missed day.
- Edge cases to test: DST changes, timezone change, completion at 23:59:59, duplicate submission.

## XP & ranks
Level score → XP. Ranks (example): Intern → Junior → Mid → Senior → Staff → Principal. Thresholds in config, not code.

## Badges (examples)
| Badge | Condition |
|---|---|
| First Blood | First in the world to solve a level |
| Root Cause | 5 levels with full quality bonus |
| No Hints | Track completed with zero hints |
| Streak 7 / 30 / 100 | Streak length |
| Log Detective | 3 logs-only levels solved |
| Regression Proof | 10 regression traps passed first try |
| Night Owl / Early Bird | Time-of-day solves (fun) |

## Leaderboards
- Global, weekly season, per tenant, per contest ([COMPETITIONS.md](COMPETITIONS.md)).
- Ties: earlier completion time wins.
- Show rank deltas, not just totals.

## Anti-gaming
- Score fixed at first successful verification; resubmits don't increase it.
- Suspiciously fast solves flagged to audit log, not auto-banned.
- Duplicate-diff detection across players (informational).
