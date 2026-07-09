# tech-stack/

Every technology used in LeadPilot — what it is, why it was chosen,
what version, and any important gotchas.

## Status: decided (2026-07-08)

Python + Claude Agent SDK + FastAPI + Postgres (Neon) + Render (Web
Service + Cron Job). Full breakdown, rationale, and what's still open
within the chosen stack: see `stack-overview.md`. Logged as Decision
022 in `decisions/README.md`, superseding Decision 005 (deferred).
Still expected to evolve as the build progresses — this isn't a
promise nothing changes, just that there's a concrete starting point
now instead of an open question blocking everything else.

## What belongs here (once decided)

- Master list of all tools, libraries, and frameworks
- Version of each dependency when added
- Why each was chosen over alternatives
- Known limitations, gotchas, configuration decisions
- What was considered and rejected, and why

## Suggested file naming

  stack-overview.md      Master list of all technologies

## Options considered (see Decision 022 for why Python won)

| Option | Notes |
|---|---|
| Python agent loop (Claude Agent SDK) — **chosen** | Large ecosystem for Google API + Slack + Twilio SDKs; Agent SDK fits the tool-calling pattern the PRD is already written in |
| Node.js agent (Claude Agent SDK or custom) | Consistent with Marc's NoiseToSignal/JS background, but Python's integration-library maturity won out for this project |
| Orchestration platform (n8n, Make, etc.) | Faster to prototype the hourly-run + tool-call shape, but less control over the security guard layer described in the PRD — rejected |

## Resolved (was "things to decide before locking this in")

- Scheduler: Render Cron Job, hourly — separate process from the
  always-on dashboard, not a continuously running loop
- State store: Postgres via Neon, not SQLite — the dashboard (Web
  Service) and batch run (Cron Job) are separate containers, so a
  local SQLite file wouldn't be visible to both; needs a real
  networked database from day one. Supports the atomic conditional
  update the approval gate depends on (Decision 021)
- Dashboard serving: FastAPI + server-rendered templates (Jinja2/htmx),
  not a separate SPA
- Hosting: Render — one Web Service, one Cron Job, shared Neon Postgres

## Still open within this stack

- Dedup/run-lock table schema (architecture/state-schema.md only has
  the contact-history log designed so far)
- Concurrency test for Decision 021's conditional update — now
  unblocked, not yet written
- Secrets management approach (Render env vars assumed, not confirmed)

## Notes

- Full detail lives in `stack-overview.md` — this file is the status
  summary, that one is the reference.
- Update `stack-overview.md` whenever a component is added, swapped,
  or upgraded, and log the reasoning as a new decision.
