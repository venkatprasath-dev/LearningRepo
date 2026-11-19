# Logging

Three separate systems. Do not mix them.

| | A. Player Debug Console | B. Admin Audit Log | C. Ops Observability |
|---|---|---|---|
| Audience | Player | Admin / instructor | You (platform operator) |
| Purpose | Learn to debug | Track who did what | Keep the platform healthy |
| Storage | In-memory ring buffer + optional file export | PostgreSQL `audit` schema, append-only | Loki/ELK + Prometheus |
| Phase | 4 | 4 | 9 |

---

## A. Player Debug Console

Players can see what their code is doing, and **choose what to track**.

### Features
- Live log stream in the UI (SSE or WebSocket), per challenge service.
- Filter by correlation ID, level, text, time.
- Request/response inspector: click a UI action → see the HTTP call and payload.
- **Watch values:** a small helper in challenge code (e.g. `Debug.track("label", value)`) pushes a labeled value to the console. Teaches "log the right thing."
- Per-session log level toggle: TRACE → ERROR.
- Export logs to a file for offline reading.

### Level-based access (log-reading is the skill)
| `log_access` | Behavior |
|---|---|
| `full` | Verbose logs, helpful messages (Levels 1-5) |
| `reduced` | Only INFO+, some noise |
| `logs_only` | No UI repro; logs are the only clue (boss/incident levels) |

### Log format (all services)
JSON, one line per event:
`timestamp, level, service, correlationId, spanId, playerId?, levelId?, message, fields{}`

### Rules
- Correlation ID created at the gateway, propagated via `X-Correlation-Id`, in MDC for every thread.
- **Never log secrets or tokens.** Mask known keys. A "secrets in logs" level teaches this on purpose, in a sandbox.
- Ring buffer capped (e.g. 5k lines/session) so logs can't exhaust memory.

---

## B. Admin Audit Log

Answers "who did what, when, from where."

### Event schema (`audit_events`, append-only)
| Field | Notes |
|---|---|
| `id`, `occurred_at` | Time in UTC |
| `actor_id`, `actor_role`, `tenant_id` | Who, and which institute |
| `action` | e.g. `LOGIN`, `LEVEL_START`, `SUBMIT`, `HINT_USED`, `SCORE_ADJUST`, `EXAM_START`, `EXAM_FINISH`, `ROLE_CHANGE`, `EXPORT` |
| `target_type`, `target_id` | Level, exam, user, etc. |
| `before`, `after` | JSON for changes (e.g. score override) |
| `ip`, `user_agent`, `request_id` | Origin |
| `prev_hash`, `hash` | Hash chain for tamper evidence |

### Admin UI
- Search/filter by user, action, date, tenant, level.
- Per-player timeline: start → hints → submits → completion.
- CSV export (itself an audited action).
- Anomaly flags: solve implausibly fast, many failed submits, diff identical to another player, login from many IPs, tab-switch bursts in exams.

### Integrity & privacy
- Writes only through `audit-service`; DB role has INSERT/SELECT only, no UPDATE/DELETE.
- Hash chain per tenant; nightly verifier job reports breaks.
- Retention configurable per tenant (default 1 year; exams longer).
- Store minimal PII (IDs, not names); IP retention limited.
- Instructors see only their tenant; super-admin sees all and every access is logged.

### Delivery
Phase 4: services call `audit-service` via REST (outbox table so failures don't lose events). Move to a broker later if volume demands.

---

## C. Ops observability (Phase 9)
Actuator health/metrics → Micrometer → Prometheus → Grafana; logs → Loki/ELK; traces → OpenTelemetry. Define SLOs (submit p95, availability) and alerts.
