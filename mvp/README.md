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

## Phase 1 MVP — derived from PRD v1.04

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

### Tools (ten required for Phase 1)

- [ ] `fetch_all_leads` — Google Sheets API via `GoogleSheetsConnector`
      (see architecture/README.md connector layer), pulls all rows
      across designated spreadsheets
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
- [ ] `verify_drive_contents` — Google Drive API, file presence/type/size
- [ ] `dispatch_slack_handoff` — Slack Web API, drafts a completion
      handoff, info request, or urgent callback request to exactly 3
      stakeholder accounts; stages only, fires after rep approval —
      replaces the retired `initiate_backoffice_call`, no exception for
      urgent messages
- [ ] `search_communications` — searches email/text history by any
      known contact identifier (email, phone, name, company)
- [ ] `update_lead_sheet` — writes a rep-approved edit back to the
      source sheet after a shown current-vs-proposed diff
- [ ] `log_call_outcome` — rep reports what happened on a call
      (answered/no answer/voicemail/didn't call); writes to the
      contact-history log; no approval token needed, since it's a
      rep-sourced fact with no external effect

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

## Phase 1 — explicitly out of scope (confirm with owners before Phase 2 planning)

- CRM integrations beyond Google Sheets
- Multi-tenant / multi-org support (single sales org for Phase 1)
- Historical analytics dashboard beyond the success metrics in the PRD
- Mobile app (dashboard is web-first)

## Notes

- Check items off only when built AND run against the eval card in
  testing/eval-suite.md — not just coded.
- The PRD (prd/LeadPilot_PRD_v1.04.md) is the authoritative source for
  full feature detail; this checklist tracks completion status only.
