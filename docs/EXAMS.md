# Institute & Exam Mode

Phase 8. Lets a college or institute host the platform and run graded debugging exams.

## Core rule
**Learning mode trusts the player's machine. Exam mode trusts nothing on it.**
- No local Ollama review, no player-run scoring.
- Student works in an institute-hosted workspace; `exam-service` grades on a locked runner against hidden tests.

## Multi-tenancy

```
Tenant (institute)
 └─ Department
     └─ Batch / Class
         └─ Student
```

| Role | Can |
|---|---|
| Super-admin (you) | Manage tenants, global level bank, platform config |
| Institute admin | Manage departments, batches, instructors, branding, SSO, data retention |
| Instructor | Build exams, assign to batches, view results, re-evaluate, export |
| Student | Take exams, view own results, practice in learning mode |

- Tenant ID on every row; enforced in a query filter layer **and** tested (Phase 8 gate: tenant A can't read tenant B).
- Isolation choice: shared schema + `tenant_id` first. Move to schema-per-tenant only if an institute demands it.
- Onboarding: bulk CSV import of students (idempotent, validation report), later SSO (Google/Microsoft/SAML).
- Branding: logo, colors, institute name on exam pages and result PDFs.
- Private level sets per tenant (their own questions stay theirs).

## Exam builder (instructor)
- Pick levels from the bank (global + tenant-private); set marks per level.
- Duration, window (start/end), attempts, access code, allowed batches.
- Options: randomize level order; **variants** (same bug class, different data/service, so neighbors can't copy); negative marking; partial credit by hidden-test pass ratio.
- Preview as student; dry-run on the runner before publishing.

## Exam session
1. Student enters with access code inside the window.
2. Server creates a session: start time, deadline, assigned variants.
3. Workspace provisioned (container per student; resource-limited; network-restricted).
4. Hints and LLM feedback disabled. Debug Console allowed (configurable: it's part of the skill).
5. Autosave of work; reconnect resumes the session; **timer is server-side**.
6. Submit or auto-submit at deadline → grading queue.

## Grading
- Hidden tests run on a locked runner against the student's final code snapshot.
- Score = Σ marks × pass ratio; configurable rounding; negative marking optional.
- Optional manual review step: instructor can read the diff, override marks (reason required, audited).
- Re-evaluation: re-run with updated tests; before/after kept.

## Integrity
| Control | Notes |
|---|---|
| Server-side timer & grading | Core |
| Variants per student | Reduces copying |
| Similarity report | Diff similarity between students; flags, not verdicts |
| Audit trail per student | Every action ([LOGGING.md](LOGGING.md)) |
| Tab-switch / fullscreen-exit events | Logged as signals; instructor decides |
| Optional proctor view | Live status board: active, idle, submitted, flags |
| No internet in workspace | Egress blocked except platform |
| Time-extension per student | Accessibility / incident; audited |

Be honest in the docs: browser-based detection is a **deterrent, not proof**. Final judgment is human.

## Results & export
- Instructor: per-student and per-level breakdown, class statistics, distribution.
- Export CSV/PDF (marks, time, flags). LMS grade passback (LTI) in Phase 10.
- Student: score, per-level result, reference solution after the exam closes (instructor-controlled).

## Hosting for institutes
- Deployment modes: (1) SaaS multi-tenant; (2) institute self-host via compose/Helm from `bughunt-deploy`.
- Capacity planning doc: runners per concurrent student (CPU/RAM/limits), peak at exam start, pre-warming.
- Data: retention policy, export, deletion on request, backup/restore.

## Data (see [DATABASE.md](DATABASE.md))
`tenants`, `departments`, `batches`, `memberships`, `exams`, `exam_levels`, `exam_sessions`, `exam_submissions`, `exam_results`, `exam_flags`.
