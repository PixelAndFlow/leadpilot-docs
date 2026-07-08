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

## Phase 1 MVP — derived from PRD v1.01

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
      handoff (call proposal or predetermined Slack message) side by
      side with a before/after diff for any spreadsheet change, and
      approves each with one click — nothing fires without that
      approval

### Tools (all seven required for Phase 1)

- [ ] `fetch_all_leads` — Google Sheets API, pulls all rows across
      designated spreadsheets
- [ ] `get_contact_history` — Google Voice API / call log tracker
- [ ] `verify_drive_contents` — Google Drive API, file presence/type/size
- [ ] `dispatch_slack_handoff` — Slack Web API, drafts a handoff or
      info-request message to exactly 3 stakeholder accounts; stages
      only, fires after rep approval
- [ ] `search_communications` *(new)* — searches email/text history by
      any known contact identifier (email, phone, name, company)
- [ ] `update_lead_sheet` *(new)* — writes a rep-approved edit back to
      the source sheet after a shown current-vs-proposed diff
- [ ] `initiate_backoffice_call` *(new)* — proposes a call to a
      back-office stakeholder as an alternative to the Slack handoff;
      stages only, places the call after rep approval

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
      without a valid, single-use rep-approval token (see
      security/threat-model.md, "autonomous execution bypass")
- [ ] Authenticated-session requirement on every agent run, data view,
      and approval (see security/threat-model.md, "unauthorized agent
      access")

### Unified interface (rep-facing)

- [ ] Prioritized queue view matching the system prompt's JSON output
      shape
- [ ] Missing-document checklist per lead
- [ ] Recommended-action cards per lead (call / text / email / info
      request) with inline draft review and edit, each requiring
      explicit approval before send — **not** auto-sent
- [ ] Back-office handoff review card (call proposal or Slack message
      draft) requiring explicit approval before it fires
- [ ] Inline spreadsheet editing with a current-vs-proposed diff shown
      before any write is confirmed
- [ ] Communications search box (search by name, company, email, or
      phone; returns history, attachments, confirmed documents)
- [ ] Login/authenticated-session gate before any of the above is
      reachable

## Phase 1 — explicitly out of scope (confirm with owners before Phase 2 planning)

- CRM integrations beyond Google Sheets
- Multi-tenant / multi-org support (single sales org for Phase 1)
- Historical analytics dashboard beyond the success metrics in the PRD
- Mobile app (dashboard is web-first)

## Notes

- Check items off only when built AND run against the eval card in
  testing/eval-suite.md — not just coded.
- The PRD (prd/LeadPilot_PRD_v1.01.md) is the authoritative source for
  full feature detail; this checklist tracks completion status only.
