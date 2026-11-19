# Levels

A level = one intentionally broken behavior + a way to prove it's fixed.

Levels are the **challenge gates** of course topics ([COURSES.md](COURSES.md)): a topic lists the level IDs it requires. A level can also be played on its own in Practice mode.
Rule: a level used as a gate must teach what its topic's lesson taught. If the lesson doesn't cover it, the gate is unfair; move it or fix the lesson.

## Tracks

| Track | Bugs taught | Example |
|---|---|---|
| 1. Basics | Type coercion, off-by-one, null handling, wrong operator | Calculator `2+2=22` |
| 2. Data & JPA | N+1, LazyInitializationException, wrong join, missing transaction, timezone | Order list loads 500 queries |
| 3. Concurrency | Race condition, deadlock, non-atomic counter, unsafe shared state | Inventory goes negative |
| 4. Distributed | Missing timeout, retry storm, non-idempotent POST, stale cache, clock skew | Duplicate payments |
| 5. Security | SQL injection, IDOR, broken auth, secrets in logs | `/orders/{id}` leaks others' data |
| 6. Performance | Slow query, missing index, memory leak, thread starvation | Heap grows until OOM |
| 7. Incidents (boss) | Logs-only, no UI repro, multiple causes | "Prod is down" scenario |
| 8. Tooling labs | Stack traces, `git bisect`, SQL `EXPLAIN`, thread/heap dumps, remote debug | Find the bad commit |

## Level spec (`levels/<id>/level.yaml`)

```yaml
id: L1-001
title: The Calculator That Concatenates
track: basics
difficulty: 1            # 1-5
estimated_minutes: 10
service: challenge-calculator
symptom: "2 + 2 shows 22"
expected: "2 + 2 shows 4"
log_access: full         # full | reduced | logs_only
hints:                   # see hints.md, 3 tiers
  - nudge
  - location
  - near_solution
tests:
  visible: calculator/visible
  hidden: server-side
rubric: rubric.md        # what Ollama checks
tags: [string-vs-number, rest]
topic: rest-types        # topic that uses this level as a gate (optional)
variants: [L1-001-b, L1-001-c]   # used for review challenges and exams
```

Files per level: `level.yaml`, `bug.patch` (applies the bug to a clean service), `hints.md`, `rubric.md`, `solution.md` (reference fix, never shipped to players), hidden tests.

## Level 1 walkthrough (design)

1. UI: calculator page. Player enters `2 + 2`, presses `=`. Shows `22`.
2. Frontend sends `POST /api/calc/add {a: 2, b: 2}`. Frontend is correct.
3. Player opens the Debug Console: sees request and response `{"result":"22"}`.
4. Player finds `CalculatorController` → `CalculatorService.add`. Operands are treated as `String` and joined with `+`.
5. Fix: parse to numeric type, add, handle invalid input.
6. UI shows `4`. `learn submit L1-001`.
7. Hidden tests also check: negatives, decimals, large values, invalid input → 400, not 500. **A naive fix that only special-cases `2+2` fails.**

Review rubric highlights: root cause fixed (not symptom), numeric type choice (`BigDecimal` vs `double`), input validation, no unrelated changes.

## Level design rules
- One primary bug; max one decoy.
- Symptom visible in UI or API within 30 seconds.
- A **regression trap** (obvious fix breaks a hidden case) appears from difficulty 3 up.
- Every level has all 3 hint tiers and a reference solution.
- Hidden tests: at least 1 for root cause, 1 for edge case, 1 for no-regression.
- Bug must be deterministic. No flaky concurrency levels without a seeded harness.

## Authoring checklist
- [ ] `bug.patch` applies cleanly to base
- [ ] Hidden tests fail on buggy, pass on reference fix
- [ ] Symptom reproducible via UI
- [ ] 3 hints, none revealing the full fix before tier 3
- [ ] Rubric lists what to accept and what to reject
- [ ] Time estimate validated by one tester
- [ ] Logs match `log_access` setting

## Hints
- Tier 1 nudge (-5% score): where to look conceptually.
- Tier 2 location (-15%): file/method.
- Tier 3 near-solution (-30%): what's wrong, not the code.
- Hints can be pre-written or Ollama-generated from the rubric; never reveal the patch.
