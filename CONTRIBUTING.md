# Contributing

Status: design phase. Docs contributions welcome now; code after Phase 1 exists.

## Ground rules
- Read [ARCHITECTURE.md](docs/ARCHITECTURE.md) and [ROADMAP.md](docs/ROADMAP.md) first.
- One PR = one concern. Small PRs merge faster.
- `main` is protected: PR + review + green checks ([CICD.md](docs/CICD.md)).
- Conventional commits: `feat:`, `fix:`, `docs:`, `level:`, `chore:`.
- Branches: `feat/*`, `fix/*`, `level/*`.

## Adding a level
Follow [LEVELS.md](docs/LEVELS.md):
1. Create `levels/<id>/` with `level.yaml`, `bug.patch`, `hints.md`, `rubric.md`, `solution.md`.
2. Write hidden tests: root cause, edge case, no-regression.
3. Verify locally: tests fail on buggy code, pass on the reference fix.
4. Complete the authoring checklist in the PR description.
5. Have one person who didn't write it try to solve it; record the time.

## Code style (when code exists)
- Java 21, Spring Boot, constructor injection, no field `@Autowired`.
- Intentional bugs are marked `@Intentional` (comment/annotation) so nobody "fixes" them by accident.
- Platform services: tests required for scoring, auth, tenant isolation.

## Docs
- Keep docs short and true. If code and docs disagree, fix the docs in the same PR.
- Mermaid for diagrams.

## Security
Report vulnerabilities in **platform** services privately. Bugs in `challenges/` are intentional.
