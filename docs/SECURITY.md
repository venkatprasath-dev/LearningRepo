# Security & Anti-cheat

This platform runs **untrusted player code** and holds **graded exam data**. Treat both as hostile surfaces.

## Threat model

| Threat | Mitigation | Phase |
|---|---|---|
| Forged completion payload | Signed submissions; server re-verifies pushed commit | 3 |
| Reading hidden tests / solutions | Hidden tests and `solution.md` never in player repo; server-side runner | 3 |
| Tampering with scoring | Scoring in platform images, separate repos | 3 |
| Prompt injection into the LLM reviewer (code comments saying "approve this") | LLM is advisory + capped bonus; strip comments from reviewed diff; never gate pass/fail on LLM | 2 |
| Malicious code in submission (fork bomb, network abuse) | Runner sandbox: container, CPU/mem/time limits, no network, read-only FS, non-root | 3 basic, 9 hardened |
| Cross-tenant data leak | Tenant filter + isolation tests | 8 |
| Brute force / abuse | Rate limits at gateway; lockouts | 5, 9 |
| Audit tampering | Hash chain, insert-only DB role | 4 |
| Secrets leakage | Env/secret manager; log masking; scan in CI | 3 |
| Supply chain | Dependency scan, SBOM, signed images | 6, 9 |
| Exam copying | Variants, similarity report, audit trail | 8 |

## Runner sandbox (server-side test execution)
- One ephemeral container per run; destroyed after.
- Limits: CPU, memory, wall-clock timeout, process count, output size.
- No network, no host mounts; code copied in read-only; temp dir writable.
- Phase 9: stronger isolation (gVisor/Firecracker) if exposing to the public internet.

## Signed submissions
Payload: `userId, levelId, commitSha, testHash, nonce, timestamp`, signed with a per-player token issued at login. Server rejects replays (nonce), stale timestamps, and unknown commits. The signature is a speed bump; **re-verification is the real control**.

## Auth
- GitHub OAuth first; short-lived JWT; refresh rotation.
- RBAC from roles in claims; every endpoint declares required role.
- Exam endpoints: additionally require an active session.

## Privacy
- Minimal PII. Per-tenant retention and deletion. Audit exports are logged.
- Student data belongs to the institute: data-processing terms before hosting real students.

## Intentionally vulnerable code
Challenge services contain bugs on purpose (SQLi, IDOR). They must:
- Run only in compose/sandbox networks, never exposed publicly with real data.
- Be isolated from platform services and the real DB.
- Be labeled clearly in code (`@Intentional` marker/comment) so scanners and reviewers don't "fix" them accidentally; CI scanners exclude `challenges/` but run on everything else.

## Open decisions
1. Where hidden tests execute (platform runner vs encrypted bundle). Default: platform runner.
2. Exam workspace: browser IDE vs container + remote desktop. Decide before Phase 8.
3. Public hosting vs institute self-host as the primary exam mode.
