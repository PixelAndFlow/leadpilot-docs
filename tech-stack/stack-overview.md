# Stack overview

Locked 2026-07-08 — see decisions/README.md Decision 022 for the
reasoning and rejected alternatives. This is the master list; update
it whenever a component is added, swapped, or upgraded.

## Runtime

- **Python 3.11+** — chosen over Node.js/Claude Agent SDK-in-TS or an
  orchestration platform (n8n/Make) for the ecosystem fit: mature,
  well-documented Google API, Twilio, and Slack SDKs, and full control
  over the tool-calling loop, which matters given how much of the PRD
  (structural-parsing security guard, exact eval-card matching) depends
  on knowing precisely what the agent did and why.

## Agent orchestration

- **Claude Agent SDK (Python)** — runs the tool-calling loop described
  in PRD system prompt v1.04: `fetch_all_leads` -> `get_contact_history`
  -> prioritize -> `verify_drive_contents` -> draft actions -> draft
  handoff. Chosen over LangChain (unnecessary abstraction weight for a
  single fixed sequence) and a fully hand-rolled loop against the raw
  Anthropic Messages API (the Agent SDK already handles the tool-call
  parsing/retry mechanics this would otherwise require reinventing).
- Tool definitions in code should mirror PRD v1.04 section 3a exactly
  — name, inputs, and the execution-gating rule per tool. If a tool's
  shape changes, that's a PRD version bump first, code change second.

## Web framework / dashboard

- **FastAPI** — serves the rep-facing dashboard and the API endpoints
  the frontend calls (queue view, approve/reject, diff view, search,
  call-outcome logging). Pydantic models double as validation for the
  system prompt's strict JSON output contract (3b OUTPUT FORMAT) —
  define that shape once as a Pydantic model and get free validation
  on every agent run's output.
- **Server-rendered templates (Jinja2) + htmx, with a little vanilla
  JS for the one browser-native interaction** (`navigator.clipboard.writeText()`
  for the `initiate_lead_call` handoff — see PRD v1.03/Decision 016).
  Chosen over a separate React app: the dashboard's job is narrow
  (list, review, approve, diff, search), and a single Python codebase
  with no separate JS build pipeline is less for a two-person team to
  maintain. Revisit if the interface outgrows what htmx handles
  comfortably.

## State store

- **Postgres, hosted on Neon.** This is the one place the original
  recommendation needed correcting: SQLite doesn't actually fit this
  topology. The dashboard (an always-on Web Service) and the batch
  agent run (a separate, periodically-invoked Cron Job) are two
  different processes in two different containers — a local SQLite
  file written by one is not visible to the other. Anything both sides
  need to read and write — the contact-history log, the approval-gate
  `stage` field (Decision 021), dedup/run-lock state — needs a real
  network-accessible database from day one, not a "SQLite now, migrate
  later" path, because the two-container split is Day 1 architecture,
  not something introduced after launch.
- Used identically in local dev (a Neon dev branch, or Postgres via
  Docker) and production, so the atomic conditional-update pattern
  (`UPDATE ... WHERE stage = 'approved'`) that the approval gate
  depends on is tested against the same engine it runs on in
  production — avoids SQLite/Postgres concurrency-semantics
  differences masking a real bug.
- Schema: see `architecture/state-schema.md` for the contact-history
  log table (which also carries the approval-gate `stage` field) and
  the dedup/run-lock tables (`lead_source_rows`, `lead_action_locks`,
  `agent_run_locks` — built Step 1, Decision 025; `agent_run_locks`
  reworked to a per-rep mutex in Step 2, Decision 027/032).

## Scheduler

- **Render Cron Job**, hourly. As of Decision 027, the job now loops
  over every rep who has connected a Google account (Decision 026),
  running the fetch/prioritize/verify/draft sequence once per rep
  using that rep's own OAuth grant — not a single global pass over one
  shared sheet list. Still no task queue (Celery/RQ) and no persistent
  worker process; at one sales org's rep count and one run per hour,
  a simple in-process loop within the same Cron Job invocation is
  enough. `agent_run_locks` (Decision 025) moved from a singleton mutex
  to a per-rep mutex to match (Decision 027/032, built Step 2) —
  `architecture/state-schema.md` has the current schema. Separate from
  the always-on Web Service; see the two-service split above.

## External integrations

- **Google (Sheets, Drive, Gmail)** — `google-api-python-client` +
  `google-auth-oauthlib`, one consistent per-rep OAuth model across
  all three (Decision 026, reversed 2026-07-11 — **supersedes the
  service-account plan from Decision 024**). Sheets and Drive both use
  the Google Picker API for consent — each rep does a one-time "Connect
  Google Account" consent, then selects which specific sheets/folders
  LeadPilot may touch via Google's own file picker, nothing pre-shared
  to a standing identity — but they don't share one scope anymore.
  `GoogleSheetsConnector` (`fetch_all_leads`/`update_lead_sheet`/
  `fetch_ad_hoc_sheet`) still uses `drive.file`. `verify_drive_contents`
  needed `drive.readonly` added on top as of **Decision 033**:
  `drive.file`'s per-item grant turned out not to extend to a granted
  folder's *contents*, confirmed live, so it couldn't do its actual job
  under `drive.file` alone. `drive.readonly` is a real, meaningfully
  bigger scope than `drive.file` — see `compliance/README.md` for the
  Google restricted-scope verification consequence this triggers, and
  Decision 033's "flagged to revisit" note. Gmail uses the same OAuth
  client for its own consent (`send_lead_email` sends *as* a specific
  rep's own Gmail account), which was always a per-rep case even under
  the old plan — see Decision 026 for the full reasoning on why
  Sheets/Drive moved to match it.
- **Twilio** (Python SDK) — `send_lead_text`. No viable free/Google
  Voice-based alternative exists (`testing/known-issues-log.md` Issue
  001) — this is a real, billed vendor relationship, not a placeholder.
- **Slack** — `slack_sdk`. Bolt's interactivity framework isn't needed
  yet; `dispatch_slack_handoff` only posts messages, it doesn't need to
  handle inbound Slack interactions (buttons, slash commands) in Phase 1.
- **Google Picker API** (new, Decision 026) — client-side JS widget
  embedded in the dashboard (Step 3) so a rep can select which specific
  sheets/folders LeadPilot may access after granting OAuth consent
  (`drive.file` + `drive.readonly` as of Decision 033). Needs its own
  API key (separate from the OAuth client secret, lower sensitivity,
  but still tracked in security/secrets-rotation-runbook.md).

## Hosting

- **Render** — one Web Service (FastAPI dashboard, always-on) and one
  Cron Job (hourly batch run), both connecting to the same Neon
  Postgres instance. Auto-deploy from GitHub on push to main.

## CI

- **GitHub Actions**, minimal to start: run the eval suite
  (`testing/eval-suite.md`) against every PR before merge. This is the
  actual regression gate the PRD's eval card is meant to be — not
  optional polish once tools exist to test.

## Known gaps / not yet decided

Updated 2026-07-16 — the four items this section used to list as open
(concurrency test, per-rep mutex schema, `rep_google_credentials`
table, `GoogleSheetsConnector` rework) were all resolved in Step 1/Step
2 and are removed below; see `decisions/README.md` Decisions 021, 025,
026, 027, 029, 032 and `mvp/README.md`'s Step 1/2 checklists for the
verified detail. One item remains genuinely open:

- Secrets management — the default assumption (Render env vars for
  static secrets; per-rep OAuth refresh tokens in Postgres, Fernet-
  encrypted per Decision 029) is implemented, but no runbook states
  this is the final, confirmed answer for production rather than a
  placeholder. See `OUTSTANDING-ITEMS.md` Tier 3.
