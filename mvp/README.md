# mvp/

MVP scope definitions and checklists for LeadPilot Phase 1. The MVP is
the smallest complete version worth running against real leads.

## What belongs here

- P0 feature/tool checklists per phase
- Clear in-scope and out-of-scope lists
- Completion tracker — check off when built AND verified against the
  eval card, not just coded

## Suggested file naming

  LeadPilot_MVP_Phase1_Checklist.md
  LeadPilot_MVP_Phase2_Checklist.md

## Phase 1 MVP — derived from PRD v1.05

### Core value props (must all be true to call Phase 1 done)

- [ ] **The Vitamin** — multi-spreadsheet aggregation cross-references
      and updates all lead tables continuously; 0% duplicate contact
      rate achieved; rep-approved edits write back to the source sheets
      from one unified interface (no per-sheet editing)
- [ ] **The Painkiller** — automated context auditing determines the
      correct outreach channel, tracks prior touchpoints, presents a
      curated missing-document checklist, and lets the rep pull a
      client's full email/text history by name, company, email, or
      phone via `search_communications`
- [ ] **The Steroid** — a single review screen where the rep sees
      drafted outreach (call/text/email) and the drafted back-office
      Slack handoff (standard, info-request, or urgent callback
      request) side by side with a before/after diff for any
      spreadsheet change, and approves each with one click — nothing
      fires without that approval; for calls, "fires" means a
      clipboard copy, not a dialed connection

### Tools (eleven required for Phase 1, updated in v1.05)

- [ ] `fetch_all_leads` — Google Sheets API via `GoogleSheetsConnector`
      (see architecture/README.md connector layer), authenticated as
      the requesting rep via per-rep OAuth (`drive.file` scope,
      Decision 026 — **not** a service account or a static admin list).
      Pulls rows only from sheets that rep personally connected via
      the Google Picker
- [ ] `get_contact_history` — reads LeadPilot's own contact-history log
      (architecture/state-schema.md); no external API, not Google Voice
- [ ] `initiate_lead_call` — stages a recommended call; on rep approval,
      copies the lead's phone number to the clipboard and shows a
      confirmation message. No telephony API, no automation of Google
      Voice or any calling app — the rep dials manually
- [ ] `send_lead_text` — drafts a text to the lead; stages only, sends
      after rep approval
- [ ] `send_lead_email` — drafts an email to the lead; stages only,
      sends after rep approval
- [ ] `verify_drive_contents` — Google Drive API, authenticated as the
      requesting rep (same `drive.file`/Picker consent as
      `fetch_all_leads`, Decision 026), file presence/type/size
- [ ] `dispatch_slack_handoff` — Slack Web API, drafts a completion
      handoff, info request, or urgent callback request to exactly 3
      stakeholder accounts; stages only, fires after rep approval —
      replaces the retired `initiate_backoffice_call`, no exception for
      urgent messages
- [ ] `search_communications` — searches email/text history by any
      known contact identifier (email, phone, name, company)
- [ ] `update_lead_sheet` — writes a rep-approved edit back to the
      source sheet after a shown current-vs-proposed diff, authenticated
      as the approving rep (Decision 026) so Google Sheets' own
      revision history attributes the edit to that specific rep
- [ ] `log_call_outcome` — rep reports what happened on a call
      (answered/no answer/voicemail/didn't call); writes to the
      contact-history log; no approval token needed, since it's a
      rep-sourced fact with no external effect
- [ ] `fetch_ad_hoc_sheet` *(new in v1.05, name provisional)* — reads a
      single sheet the rep points LeadPilot at mid-session, via the
      same per-rep `drive.file`/Picker consent, for a one-off lookup
      outside the routine hourly scan; read-only, no approval token
      needed (Decision 028)

### Prioritization logic

- [ ] Rank 1: active interest within last 24 hours
- [ ] Rank 2: new uncontacted leads
- [ ] Rank 3: old leads needing multi-channel cadence follow-up

### Security guard (non-negotiable for Phase 1 — see security/)

- [ ] Isolated validation layer strips injection keywords before any
      tool call executes
- [ ] Atomic state locking commits run timestamps before tool calls
      are authorized (prevents duplicate contact)
- [ ] File size/type checkpoint (>5KB, strict PDF extension) before a
      document is counted as present
- [ ] Execution gate: every side-effect tool call rejects execution
      unless that action's log row is in `approved` state, enforced via
      a single atomic conditional update — no separate token object
      (see architecture/state-schema.md and security/threat-model.md,
      "autonomous execution bypass")
- [ ] Authenticated-session requirement on every agent run, data view,
      and approval (see security/threat-model.md, "unauthorized agent
      access")
- [ ] Data access guard (new in v1.05, Decision 026): `fetch_all_leads`,
      `verify_drive_contents`, and `fetch_ad_hoc_sheet` only ever use
      the currently authenticated rep's own stored Google credential —
      never another rep's, never a shared/standing credential (see
      PRD v1.05 Eval Case 11)

### Unified interface (rep-facing)

- [ ] Prioritized queue view matching the system prompt's JSON output
      shape
- [ ] Missing-document checklist per lead
- [ ] Recommended-action cards per lead (call / text / email / info
      request) with inline draft review and edit, each requiring
      explicit approval before send — **not** auto-sent; approving a
      call copies the number to the clipboard and shows a confirmation
      message, it does not dial or open any calling app
- [ ] Back-office handoff review card (Slack message draft — standard,
      info request, or urgent callback request) requiring explicit
      approval before it posts, no exception for urgent messages
- [ ] Inline spreadsheet editing with a current-vs-proposed diff shown
      before any write is confirmed
- [ ] Communications search box (search by name, company, email, or
      phone; returns history, attachments, confirmed documents)
- [ ] Call-outcome quick action (answered / no answer / voicemail /
      didn't call) so Rank 3's unanswered-call follow-up rule has data
      to run on — no approval token needed, this is the rep reporting
      a fact
- [ ] Login/authenticated-session gate before any of the above is
      reachable
- [ ] "Connect Google Account" flow (new in v1.05, Decision 026) — a
      one-time OAuth consent (`drive.file` scope) plus the Google
      Picker widget so the rep selects which sheets/folders LeadPilot
      may access; also the entry point for `fetch_ad_hoc_sheet` when a
      rep hands LeadPilot a sheet it hasn't seen yet

## Build order (added 2026-07-08, now that the stack is locked — Decision 022)

Sequenced so each step only depends on what's already done above it —
build in this order, not by feature preference. Uses "Step," not
"Phase," to avoid confusion with the Phase 1/Phase 2 product-scope
language used elsewhere in this file and in roadmap/README.md — this
is a build sequence within Phase 1, not a different product phase.

### Step 0 — accounts and access (no code yet)

Fully done 2026-07-11 by Marc. All five items below are real accounts
with real credentials obtained, not just planned — see each item for
specifics.

- [x] ~~Google Cloud project + a service account (JSON key)~~ — done
      2026-07-11 by Marc, but this access model (Decision 024) was
      **superseded the same day by Decision 026**. The project and
      service account created here still back the existing Step 1
      `GoogleSheetsConnector` for local dev (see Step 1 below) until
      Step 2 reworks it — don't delete them yet, but new work should
      follow the item below instead
- [x] Google Cloud project (already created above) + OAuth consent
      screen configured, requesting the `drive.file` scope (Sheets and
      Drive) and the Gmail send scope (Step 2's `send_lead_email`) —
      **not** a service account (Decision 026, reverses Decision 024).
      Created one OAuth client ID/secret (`LeadPilot Web`, web
      application type) covering all three. Sheets API, Drive API, and
      Gmail API all enabled on the project. Done 2026-07-11 by Marc —
      external user type, test users added (Marc + Abdoul) since the
      app stays in Testing publishing status for now, not submitted
      for Google's verification
- [x] Google Picker API enabled on the same project (`LeadPilot_Google_Picker`
      API key, restricted to the Picker API only and to
      `http://localhost:8000/*` as an allowed website referrer for
      now) — this is what lets a rep pick specific sheets/folders to
      grant LeadPilot, rather than anything being pre-shared (Decision
      026). Done 2026-07-11 by Marc
- [x] Twilio account + phone number — free trial account, trial number
      auto-assigned (no purchase needed). Account SID and Auth Token
      obtained from the console dashboard. Trial limitation to note
      for Step 2 testing: can only call/text verified numbers until
      the account is upgraded
- [x] Slack app registered with `chat.postMessage` scope, installed
      where the 3 back-office stakeholders are — app created
      (`LeadPilot`), `chat:write` bot token scope added, installed to
      the workspace, Bot User OAuth Token (`xoxb-...`) obtained. Real
      3-stakeholder channel/DM list not finalized yet (business
      decision, not a Step 0 blocker) — a single test channel was
      created and its channel ID captured as a placeholder for local
      testing
- [x] Neon Postgres project (dev + prod) — project `leadpilot` created;
      dev/prod split done via Neon's branching feature (a `dev` branch
      off the default/production branch) rather than two separate
      projects, so both share one project's infra. Connection strings
      obtained for both branches
- [x] Render account, GitHub repo connected for auto-deploy — signed
      up via GitHub, Render's GitHub App already had sufficient access
      to `abdoulk30/LeadPilot` with no extra approval needed from
      Abdoul (confirmed by reaching the "New Web Service" source-code
      screen, which showed the repo connected with Python auto-detected
      and `main` pre-selected). Project `LeadPilot` created with a
      `Production` environment, ready for Step 4 — the actual Web
      Service/Cron Job aren't created yet, intentionally, since there's
      no real functionality to deploy until Step 2

### Step 1 — foundation (no product logic yet)

Done 2026-07-10 by Abdoul, on `abdouls-branch` in the `leadpilot`
code repo; merged to `main` by Marc 2026-07-10 (commit `cc4c8ac`).
All four
items below are real, working code with passing tests against a real
local Postgres (and the connector against a real live Google Sheet),
not just designed on paper — see each item's test file for evidence,
per this file's own "check off only when built AND verified" rule.

- [x] Contact-history + approval-gate table, from
      architecture/state-schema.md's schema — `leadpilot/gate.py`,
      tested in `tests/test_gate.py` including a concurrency test
      (10 simultaneous execute attempts on the same approved row,
      exactly one wins) that resolves the test flagged as unwritten in
      Decision 021 and Issue 003
- [x] Dedup/run-lock table — designed as three tables:
      `lead_source_rows` (raw-row-to-canonical-lead mapping),
      `lead_action_locks` (per-lead duplicate-contact prevention), and
      `agent_run_locks` (hourly-cron mutex with stale-lock recovery).
      `leadpilot/locks.py`, tested in `tests/test_locks.py` including
      concurrency tests for both lock types
- [x] Authenticated rep-session system (Decision 013) — resolved as
      email+password with our own DB-backed sessions, not Google
      OAuth, to keep this independent of Step 0's Google Cloud
      project (confirmed with Abdoul 2026-07-09 — see the new decision
      logged below). `leadpilot/auth.py` + `leadpilot/app.py`, tested
      in `tests/test_auth.py` and `tests/test_app.py` (real HTTP
      login/logout/whoami cycle, expired/revoked/deactivated-session
      rejection)
- [x] `GoogleSheetsConnector` implementing the `LeadSourceConnector`
      interface (PRD v1.04 section 3e) — not a direct Sheets API call
      from business logic. Authenticates via a Google service account
      rather than the `GOOGLE_OAUTH_CLIENT_ID`/`SECRET` flow
      `commands/README.md` originally assumed for all Google access —
      confirmed by Marc 2026-07-11 (Decision 024), **then superseded
      the same day by Decision 026**: this connector needs to be
      reworked in Step 2 to authenticate per rep via OAuth (`drive.file`
      scope) instead of one shared service account. Still real, tested
      code as shipped — the rework is additive scope for Step 2, not a
      sign anything here was wrong for what it was built against.
      Verified against a real live test spreadsheet, not mocks, in
      `tests/test_google_sheets_connector_live.py` — this test is also
      written against the service-account model (single global
      `source_id`) and will need reworking alongside the connector for
      the same reason

### Step 2 — the tools

- [ ] Implement each of the 11 tools (PRD v1.05 section 3a) one at a
      time, checked against its eval case as it's built, not after
- [ ] Rework `GoogleSheetsConnector` from the shared service account
      (Decision 024) to per-rep OAuth (`drive.file` scope, Decision
      026) — including reworking `tests/test_google_sheets_connector_live.py`,
      which currently assumes one global `source_id`
- [ ] Build `verify_drive_contents` against the same per-rep OAuth
      model from the start (Decision 026) — no service-account version
      to retrofit later, unlike Sheets
- [ ] Build `fetch_ad_hoc_sheet` (Decision 028) — finalize its name and
      signature with Abdoul, wire it to the same rep-scoped connector
- [ ] Design the `agent_run_locks` per-rep mutex (Decision 027,
      updates Decision 025's singleton design) before the batch loop
      goes per-rep
- [ ] Build the `rep_google_credentials` table (shape sketched in
      `architecture/state-schema.md`) including the encryption-at-rest
      approach for stored refresh tokens
- [ ] Prompt-injection validation layer (Decision 006)

### Step 3 — the interface

- [ ] Prioritized queue view
- [ ] Approve/reject flow wired to the status-flip mechanism
      (Decision 021) — this is where "approval" actually becomes real
- [ ] Spreadsheet diff view (current-vs-proposed)
- [ ] Communications search box
- [ ] Call-outcome quick action — UI flow not designed yet, design it
      here (Decision 020's open item)
- [ ] Back-office handoff review card (all three message types)

### Step 4 — wire it together and test

- [ ] Render Cron Job running the full system-prompt sequence hourly
- [ ] All 11 `testing/eval-suite.md` cases passing against the real
      implementation, not just designed on paper
- [ ] Concurrency test for the approval-gate conditional update
      (Decision 021's open item, unblocked now that Postgres is chosen)

### Step 5 — before this touches a real lead

- [ ] TCPA/CAN-SPAM/do-not-call compliance work (compliance/README.md)
- [ ] `search_communications` compliance/retention review
      (testing/known-issues-log.md Issue 004)
- [ ] Real written partnership agreement (governance/README.md —
      50/50 and fork rights are confirmed verbally, not yet on paper)
- [ ] Security review against security/threat-model.md and
      security/pen-test-checklist.md
- [ ] Soft launch: one rep, one sales org, before opening it up further

## Phase 1 — explicitly out of scope (confirm with owners before Phase 2 planning)

- CRM integrations beyond Google Sheets
- Multi-tenant / multi-org support (single sales org for Phase 1)
- Historical analytics dashboard beyond the success metrics in the PRD
- Mobile app (dashboard is web-first)

## Notes

- Check items off only when built AND run against the eval card in
  testing/eval-suite.md — not just coded.
- The PRD (prd/LeadPilot_PRD_v1.05.md) is the authoritative source for
  full feature detail; this checklist tracks completion status only.
