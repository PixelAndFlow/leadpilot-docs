# LeadPilot — Context File v003

Paste this at the start of a new AI session to restore full context.
Supersedes v002 (which was stale — still described 7 tools, PRD v1.01,
and "tech stack not yet chosen"; v001/v002 kept unedited for
reference). This version reflects state as of 2026-07-13.

## What LeadPilot is

An AI agent (Claude Agent SDK tool-calling loop) for B2B Sales and
Business Development teams that orchestrates lead triage and
multi-channel communication pipelines across Google Workspace (Sheets,
Drive, Gmail) and Slack/Twilio. Runs on an hourly Cron Job per
connected rep. Owners: Marc Delsoin, Abdoul Ba. Never sends, calls,
texts, or writes anything without explicit rep approval first.

## What changed since v002 (PRD v1.01 → v1.06)

- **Tech stack locked** (Decision 022): Python/FastAPI backend,
  Postgres (Neon in prod, local `scripts/devdb.sh` in dev),
  server-rendered Jinja2 + htmx for the dashboard — explicitly not a
  React SPA. Deployed on Render (Web Service + Cron Job, one shared
  `DATABASE_URL`).
- **Per-rep OAuth replaces the shared service account** (Decision 026,
  supersedes Decision 024) — each rep connects their own Google
  account (`drive.file` for Picker-granted sheets, `drive.readonly`
  for reading a granted folder's contents per Decision 033,
  `gmail.send`/`gmail.readonly` for email tools). No standing shared
  credential anywhere. Refresh tokens encrypted at rest via Fernet
  (Decision 029).
- **Tool count grew from 7 to 11** (PRD v1.05): `initiate_lead_call`
  (clipboard handoff, no telephony API — Decision 016) and
  `send_lead_text` split out from what v002 called
  `initiate_backoffice_call`/generic outreach; `send_lead_email` added;
  `log_call_outcome` added (rep self-reports call outcome — closes the
  visibility gap left by Google Voice having no usable API, Issue
  001); `fetch_ad_hoc_sheet` added (one-off sheet lookup mid-session,
  Decision 028); `initiate_backoffice_call` itself retired (Decision
  019) — back-office handoff is Slack-only now via
  `dispatch_slack_handoff`, no exception for urgent messages.
- **Approval-gate mechanism concretized** (Decision 021): no separate
  token object — a `contact_history.stage` field
  (`drafted → awaiting_rep_approval → approved → executed/rejected/expired`)
  is the entire enforcement mechanism, flipped via a single atomic
  conditional `UPDATE ... WHERE stage = <expected>`. Proven under real
  concurrency (10 simultaneous execute attempts on one approved row,
  exactly one wins).
- **Two-repo split formalized**: `leadpilot` (code, this build) and
  `leadpilot-docs` (private planning/decisions, sibling repo) — see
  that repo's README "Why two repos."

Full detail: `prd/LeadPilot_PRD_v1.06.md`, `decisions/README.md`
(through Decision 033).

## The problem it solves

Outbound sales reps lose ~2.5 hours/day cross-referencing scattered
lead spreadsheets and contact histories to avoid duplicate outreach,
and deal handoffs to back-office teams are delayed ~4 hours because
notifying the 3 relevant stakeholders on Slack is manual.

## The eleven tools

Read-only / no-approval-needed tools: `fetch_all_leads`,
`fetch_ad_hoc_sheet`, `verify_drive_contents`, `get_contact_history`,
`search_communications`, `log_call_outcome` (rep-reported fact, no
external side effect to gate).

Staged-then-approved tools (each: a `run()`/staging function that only
ever calls `gate.create_draft()`, plus a separate `execute()` called
after rep approval): `initiate_lead_call` (clipboard copy only, no
telephony API — the rep dials manually), `send_lead_text` (Twilio),
`send_lead_email` (Gmail, sent via the *approving* rep's own OAuth
token), `dispatch_slack_handoff` (Slack, one of `completion_handoff` /
`info_request` / `urgent_callback_request` — no exception to the
approval gate for urgent ones), `update_lead_sheet` (writes back to
the source Google Sheet after a shown current-vs-proposed diff,
authenticated as the approving rep so Sheets' own revision history
attributes it correctly).

## Prioritization logic (system prompt v1.05, 10-step sequence)

- Rank 1: active interest within last 24 hours
- Rank 2: new uncontacted leads
- Rank 3: old leads needing multi-channel cadence follow-up
- After ranking: verify Drive contents (application, 3 months bank
  statements, prequal questionnaire) via `verify_drive_contents`; draft
  the recommended next action per lead as `AWAITING_REP_APPROVAL`; if
  docs are complete, draft a back-office Slack handoff also
  `AWAITING_REP_APPROVAL`. EXECUTION GUARD: "urgency is never a reason
  to skip rep approval." Output shape is a fixed JSON contract
  (`prioritized_queue` + `pending_backoffice_handoffs`) — see
  `context-files/leadpilot_interface_design_context-v001.md` for the
  full schema, since this is what the Step 3 dashboard binds to.

## Security posture

Primary threat: indirect prompt injection via malicious text in a
spreadsheet cell attempting to trigger unauthorized tool calls or leak
contact history. Safeguard: an isolated, non-LLM validation layer
strips instruction-like keywords from all spreadsheet-sourced text
before any tool executes — **not yet built** (Decision 006, cross-cutting,
unowned by either tool group as of this writing).

Other named failure modes: duplicate contact via race conditions
(mitigated — atomic `lead_action_locks`/`agent_run_locks`, both
INSERT...ON CONFLICT, not SELECT-then-UPDATE), false-positive file
completeness (file size >5KB + strict PDF extension checks),
autonomous execution bypass (the approval-gate state machine described
above — no separate token object), unauthorized agent access
(DB-backed rep sessions, `require_rep()` as the enforcement point,
revocable immediately unlike a stateless token).

Full detail: `security/threat-model.md`, `pen-test-checklist.md`,
`incident-response-plan.md`.

## Build status (as of 2026-07-13)

- **Step 0 (accounts/access):** complete.
- **Step 1 (foundation):** complete, merged to `main` — auth/sessions,
  approval-gate state machine, dedup/run-lock tables,
  `GoogleSheetsConnector` (originally service-account, since reworked).
- **Step 2 (the 11 tools):** complete, both groups. Group A (Abdoul,
  5 tools: `fetch_all_leads`, `fetch_ad_hoc_sheet`, `update_lead_sheet`,
  `verify_drive_contents`, `log_call_outcome`) — built, tested, and
  live-verified end to end. Group B (Marc, 6 tools:
  `get_contact_history`, `initiate_lead_call`, `send_lead_email`,
  `dispatch_slack_handoff`, `search_communications`, `send_lead_text`)
  — built and tested (57 tests against real local Postgres + fake
  injectable API clients), **not yet live-verified**: needs Marc to
  complete the live "Connect Google Account" OAuth consent flow
  (planned, not yet run), and `send_lead_text`/`search_communications`'s
  Twilio paths are blocked on a live account issue (Issue 005 below),
  not a code problem.
- **Not yet started:** the prompt-injection validation layer, Step 3
  (the interface — see the dedicated interface-design context file),
  Step 4 (wiring/eval-suite pass), Step 5 (compliance/security review
  before touching real leads).

## Open issues (see `testing/known-issues-log.md` for full log)

- **Issue 004 (open):** `search_communications` reads actual message
  content/attachments, not just metadata — needs a compliance/retention
  review before use against real client data.
- **Issue 005 (open):** Marc's Twilio trial credentials authenticate
  fine against the base Account resource but return `401 Policy
  evaluation failed` on `IncomingPhoneNumbers`/`OutgoingCallerIds`
  specifically — reproduced on two networks, ruling out local
  network/firewall interference. Root cause unconfirmed; Marc is
  waiting on a Twilio support callback. Does not block the two tools
  built against it, which are code-complete and fake-client-tested —
  only blocks live verification.

## Open decisions (see `decisions/README.md` "Open decisions")

- `contact_history` needs a real `message_type` column —
  `dispatch_slack_handoff` currently stores it in the `note` field as
  a documented stopgap. Needs Abdoul's input before Marc writes the
  migration: both the schema question itself, and migration-ordering,
  since a new migration would target the same current head as
  Abdoul's `agent_run_locks` migration.

## Known code-repo drift

The `leadpilot` code repo's own `CLAUDE.md` may lag this docs repo —
check it directly rather than trusting a cached description (this
file included). As of this writing its "Current build state" section
was stale (still listed all of Group B's tools as "not yet started"
after they were actually completed) and has been corrected in this
pass; verify it's still accurate whenever this context file is reused.

## Docs repo structure

`leadpilot-docs` (private, sibling to the `leadpilot` code repo):
`prd/`, `mvp/`, `architecture/` (incl. `state-schema.md`),
`tech-stack/`, `commands/`, `decisions/`, `research/`, `compliance/`,
`security/` (threat-model, incident-response-plan,
secrets-rotation-runbook, pen-test-checklist), `settings/`, `testing/`
(eval-suite, ci-strategy, definition-of-done, known-issues-log),
`governance/`, `roadmap/`, `release-process/`,
`monitoring-observability/`, `public/`, `ai-conversations/`,
`context-files/`, `installation/`.

## Other reusable context files in this folder

- `leadpilot_interface_design_context-v001.md` — scoped narrowly for
  handing off Step 3 (dashboard) design work to a design-focused LLM:
  screen list, real tool output shapes, the approval-gate interaction
  pattern, and the htmx/Jinja2 tech constraint.

## Success metrics (PRD v1.06, unchanged from v002)

- Admin time per rep: <20 min/day (85% reduction from current ~2.5
  hours)
- Document-arrival-to-staged-handoff latency: <60 seconds
- Rep adherence to recommended prioritization: >90%
- Prompt injection block rate: 100% (0% execution leakage)
- Side-effect actions executed without rep approval: 0%
- Agent runs/data views/approvals outside an authenticated session: 0%

## Key principle carried over from NoiseToSignal

Evidence-before-fix: a fix or feature is only "done" when you can
point to actual output proving it works (a passing test against real
Postgres, a real live API round-trip, the literal JSON payload) —
never "it looks right" or "the code should work." This session's
practice: every "complete" claim above is backed by a specific test
file and count, not an assumption.
