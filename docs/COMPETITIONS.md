# Competitions

Phase 7. Builds on scoring ([SCORING.md](SCORING.md)) and leaderboard-service.

## Contest types

| Type | Description |
|---|---|
| Daily challenge | One level/day, global board, streak credit |
| Weekly season | Cumulative XP across the week; resets Monday UTC |
| Timed contest | Fixed window (e.g. 2h), N levels, ranked by points then time |
| Incident race | One logs-only boss level; first correct fix wins |
| Team contest | 2-4 players, shared score, one submission per level per team |
| Private contest | Invite code; for clubs, classes, companies |

## Rules (configurable per contest)
- Start/end time, duration per participant (optional "virtual start").
- Level set (fixed, or random from pool by difficulty).
- Hints on/off, penalty override.
- Max attempts per level.
- Scoreboard **freeze** for the last X minutes; reveal at end.
- Tie-break: points → fewer penalty minutes → earlier last accepted submission.

## Scoreboard
- Live board via SSE; per-level cells show first-blood marker, attempts, time.
- Spectator mode (no login) for public contests.
- Tenant-scoped boards for institutes.

## Teams
- Create/join by code. Captain manages roster before start; locked after.
- Team score = sum of best accepted submission per level.

## Fairness
- Submissions verified server-side only.
- Rate limit submissions per participant per level.
- Similarity detection between participants; flagged cases reviewed by organizers, not auto-disqualified.
- Organizer can void a submission (audited, reason required).

## After the contest
- Editorial page per level: root cause, reference fix, common wrong fixes.
- Replay of top solves (diff timeline).
- Certificates (PDF) for top N / participation (optional).

## Data (see [DATABASE.md](DATABASE.md))
`contests`, `contest_levels`, `contest_participants`, `teams`, `team_members`, `contest_submissions`, `scoreboard_snapshots`.
