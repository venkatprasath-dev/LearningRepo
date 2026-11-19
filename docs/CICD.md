# CI/CD: grow the pipeline with the app

Start with the smallest pipeline that protects you. Add one stage per phase.

## Branch & PR rules (Phase 0)
- `main` is always releasable. Protected: PR + 1 review + green checks.
- Branches: `feat/*`, `fix/*`, `level/*`. Conventional commits.
- PR template: what, why, how tested, level impact.

## Pipeline evolution

| Phase | Add to pipeline | Gate |
|---|---|---|
| 0 | Markdown lint, link check | Docs valid |
| 1 | Per-service build + unit tests (GitHub Actions) | Build green |
| 1 | **Level check:** hidden test FAILS on buggy code, PASSES with reference fix | Level is real and solvable |
| 2 | CLI build + release artifact; level validation matrix over all levels | No broken level merges |
| 3 | Flyway migration check on a clean DB; Testcontainers integration tests; build + push Docker images to GHCR; compose smoke test | Images start, `/health` OK |
| 4 | Log-schema contract test; audit-immutability test; auto-deploy `main` → staging | Staging up to date |
| 5 | Scoring unit + property tests; Playwright e2e (Level 1 happy path); coverage gate (e.g. 70% on scoring/progress) | Business rules protected |
| 6 | Level-pack matrix; auto-generate level catalog; dependency scan (Dependabot/OWASP); SAST (CodeQL) | No known-vuln deps |
| 7 | k6 load test on submit/scoreboard; canary deploy | p95 latency budget |
| 8 | Tenant-isolation tests; manual security-review gate; blue/green deploy; backup/restore drill | No cross-tenant leak |
| 9 | SBOM, image signing (cosign), Helm/K8s deploy, chaos/failure tests, SLO alerts | Supply chain + resilience |
| 10 | Per-language CI templates; semver release automation | Polyglot parity |

## Environments

| Env | Purpose | Deploy |
|---|---|---|
| local | Dev + players | `docker-compose up` |
| staging | Integration, demos | Auto on merge to `main` (Phase 4+) |
| prod | Real users / institutes | Manual approval; blue/green (Phase 8+) |

## Secrets
- GitHub Actions secrets / environment secrets. Never in repo, never in images.
- Per-tenant secrets (SSO) encrypted in DB (Phase 9).

## Rollback
- Immutable image tags (git SHA). Rollback = redeploy previous tag.
- DB migrations: forward-only, backward-compatible (expand → migrate → contract).

## Rules that save time
- Fail fast: lint + unit first, integration after.
- Cache Maven/npm.
- One reusable workflow per language; services call it.
- A flaky test is a bug. Fix or quarantine within a week.
