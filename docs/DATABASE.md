# Database

PostgreSQL. One schema per service. No cross-schema joins; reference by ID. Flyway migrations per service.

Conventions: `uuid` PKs, `created_at`/`updated_at` (UTC `timestamptz`), `tenant_id` on tenant-owned rows (Phase 8+), soft-delete only where stated.

## `auth` (Phase 3)
- `users`: id, github_id, handle, email, timezone, status
- `roles`, `user_roles`
- `sessions` / refresh tokens
- (Phase 8) `tenants`, `departments`, `batches`, `memberships`

## `course` (Phase 3, 6, 8)
Curriculum content lives in files (`courses/`) and is synced into the DB on deploy; the DB stores references and learner state.
- `courses`: id, title, version, active
- `modules`, `topics`: id, course_id, order, pass_mark, test_out_allowed
- `topic_prerequisites`: topic_id, requires_topic_id (validated acyclic in CI)
- `topic_levels`: topic_id, level_id, required (bool)
- `enrollments`: user_id, course_id, enrolled_at, completed_at, certificate_id
- `topic_progress`: user_id, topic_id, state (`locked|available|learning|challenge|completed`), lesson_read_at, quiz_best_score, completed_at, completed_via (`normal|test_out`)
- `quiz_attempts`: user_id, topic_id, answers (jsonb), score, taken_at
- `review_schedule`: user_id, topic_id, due_at, interval_days (spaced repetition)
- `certificates`: id (public verifiable), user_id, course_id, issued_at
- (Phase 8) `cohort_assignments`: batch_id, course_id, start_at, due_at, drip_rules (jsonb)

`topic_progress.state` changes only via `course-service` from verified events, never from client input.

## `progress` (Phase 3-5)
- `levels`: id (e.g. L1-001), track, difficulty, estimated_minutes, version, active
- `attempts`: id, user_id, level_id, commit_sha, test_result_hash, llm_verdict (jsonb), verified (bool), score_components (jsonb), started_at, finished_at
- `level_completions`: user_id, level_id, attempt_id, score, completed_at (unique user+level)
- `hints_used`: user_id, level_id, tier, used_at
- `streaks`: user_id, current, longest, last_day, freezes
- `badges`, `user_badges`
- `xp_ledger`: user_id, delta, reason, ref_id (append-only; XP = sum)
- `daily_challenges`: day, level_id

## `leaderboard` (Phase 5, 7)
- `seasons`
- `season_scores`: season_id, user_id, points
- `contests`, `contest_levels`, `contest_participants`, `teams`, `team_members`, `contest_submissions`, `scoreboard_snapshots`

## `exam` (Phase 8)
- `exams`, `exam_levels` (marks, order), `exam_sessions` (deadline, variant map), `exam_submissions` (snapshot ref, graded_at), `exam_results`, `exam_flags`

## `audit` (Phase 4)
- `audit_events` (see [LOGGING.md](LOGGING.md)): append-only, hash chain, INSERT/SELECT-only DB role
- `audit_chain_checks`: verifier results

## Key constraints
- `level_completions` unique (user_id, level_id) → resubmits can't inflate score.
- `attempts.verified` set only by the verifier, never from client payload.
- `xp_ledger` and `audit_events` never updated or deleted.
- Index: `attempts(user_id, level_id)`, `audit_events(tenant_id, occurred_at)`, `season_scores(season_id, points desc)`.

## Local dev
Compose Postgres with a volume; seed script for levels and a demo player (when code is authorized).
