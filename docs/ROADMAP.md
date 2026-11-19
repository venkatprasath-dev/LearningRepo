# Roadmap: working app → perfect app

Rule: **every phase ends with something that runs.** Never start phase N+1 until phase N's exit criteria pass.
Do not build features you can't demo.

CI/CD detail per phase: [CICD.md](CICD.md).

| # | Phase | Goal |
|---|---|---|
| 0 | Foundations | Conventions and docs |
| 1 | Walking skeleton | Level 1 + a 2-topic roadmap work end-to-end by hand |
| 2 | Local submit + AI review | `learn` CLI + Ollama review |
| 3 | Persistence, progress & course engine | PostgreSQL, accounts, server-side topic gating |
| 4 | Logging | Player Debug Console + admin audit |
| 5 | Gamification | Score, streaks, hints, badges, leaderboard |
| 6 | Content scale | 3+ courses, 20+ levels, authoring kit |
| 7 | Competitions | Contests, teams, seasons |
| 8 | Institute & exam mode | Multi-tenant, exams, grading |
| 9 | Hardening | Anti-cheat, sandbox, observability, SSO |
| 10 | Polyglot & ecosystem | Node/Python tracks, LMS, plugins |

---

## Phase 0 - Foundations
- Scope: repo layout, level spec format ([LEVELS.md](LEVELS.md)), branch strategy, PR template, CODEOWNERS.
- Exit: a new contributor can read docs and know where a level lives.

## Phase 1 - Walking skeleton (the MVP)
- Scope: React calculator page → gateway (or direct) → `challenge-calculator` where `+` concatenates strings (`2+2=22`). Player fixes by hand. Tests included.
- Also: a **static roadmap page** reading `courses/*.yaml` with 1 course, 2 topics: lesson (markdown) → challenge → "next topic unlocks" (state kept in browser localStorage; no backend yet). This proves the course → gate → unlock loop early. See [COURSES.md](COURSES.md).
- Not in scope: DB, auth, CLI, LLM, server-side gating.
- Exit: UI shows 22 on buggy branch, 4 after fix; test fails before / passes after; Topic 2 unlocks only after Topic 1's challenge passes (client-side for now).

## Phase 2 - Local submit + AI review
- Scope: `learn doctor` (Java/Docker/Ollama/git/fork remote check), `learn test`, `learn review` (Ollama JSON verdict), `learn submit` (commit + push to fork). Results saved to a local JSON file.
- Review rubric: correctness, root cause vs symptom patch, side effects, readability. Models: `gemma4:12b` default, `mistral` fallback; tests-only if Ollama is down.
- Exit: fix Level 1 → `learn submit` → commit on fork with review summary.

## Phase 3 - Persistence, progress & course engine
- Scope: PostgreSQL, `auth-service` (GitHub OAuth), `progress-service` (levels, attempts, completions), signed submissions, server-side re-verification. Split platform services into own repos here.
- **Course engine:** `course-service` loads the curriculum from `courses/`, stores enrollments and topic states, enforces unlock rules (linear or prerequisites), quiz pass marks and challenge gates **server-side**. Roadmap view reads state from the API.
- Exit: completion appears in DB only after server verifies the pushed commit. Forged payload is rejected. A topic cannot be skipped by calling the API directly; next topic unlocks only after server-verified completion.

## Phase 4 - Logging
- Scope: structured JSON logs, correlation IDs, player Debug Console, `audit-service` + admin viewer. See [LOGGING.md](LOGGING.md).
- Exit: player traces one request across 2 services by ID in the UI; admin sees a player's timeline; audit rows can't be updated/deleted.

## Phase 5 - Gamification
- Scope: scoring ([SCORING.md](SCORING.md)), streaks + freeze, 3-tier hints, badges, XP/ranks, leaderboard, daily challenge.
- Exit: two accounts compete on a leaderboard; hint use lowers score; streak survives midnight edge cases (timezone tested).

## Phase 6 - Content scale
- Scope: 20+ levels across tracks, 3+ full courses with quizzes, authoring kit for courses and levels, community PR flow, course/level catalog, test-out and placement tests, review challenges (spaced repetition), certificates, replay of a solve.
- Exit: a contributor adds a topic (lesson + quiz + level) using only the docs; CI validates the curriculum (no prerequisite cycles, levels exist and are solvable).

## Phase 7 - Competitions
- Scope: contests, teams, scoreboard freeze, seasons ([COMPETITIONS.md](COMPETITIONS.md)).
- Exit: 50 simulated users finish a timed contest; board correct under load.

## Phase 8 - Institute & exam mode
- Scope: multi-tenancy, roles, bulk import, exam builder (from a course's topics/levels), locked runner, server-side grading, exports ([EXAMS.md](EXAMS.md)). Cohorts: instructor assigns a course to a batch with deadlines and drip release; progress dashboard.
- Exit: a pilot class of 30 completes a course section and a proctored exam; tenant A cannot see tenant B data (tested).

## Phase 9 - Hardening
- Scope: sandboxed runners, rate limits, anti-cheat signals, Prometheus/Grafana/Loki/OTel, SSO (Google/Microsoft/SAML), accessibility audit, backups ([SECURITY.md](SECURITY.md)).
- Exit: SLOs defined and alerting; restore drill passed; external-style security review done.

## Phase 10 - Polyglot & ecosystem
- Scope: Node/Python challenge tracks, LMS integration (LTI), hosted leaderboard, plugin API, semver releases.
- Exit: a non-Java level ships through the same pipeline unchanged.

---

## Scope guards
- Skip Kubernetes until Phase 9. Compose is enough before that.
- Skip a message broker until Phase 7. Use outbox + REST.
- Skip microfrontends. One React app.
- If a phase slips, cut scope, not exit criteria.
