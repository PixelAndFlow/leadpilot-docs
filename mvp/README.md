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

## Phase 1 MVP — derived from PRD v1

### Core value props (must all be true to call Phase 1 done)

- [ ] **The Vitamin** — multi-spreadsheet aggregation cross-references
      and updates all lead tables continuously; 0% duplicate contact
      rate achieved
- [ ] **The Painkiller** — automated context auditing determines the
      correct outreach channel, tracks prior touchpoints, and
      presents a curated missing-document checklist
- [ ] **The Steroid** — one-click delivery of tailored outreach
      scripts plus automatic concurrent Slack notification to all 3
      back-office stakeholders when a file becomes complete

### Tools (all four required for Phase 1)

- [ ] `fetch_all_leads` — Google Sheets API, pulls all rows across
      designated spreadsheets
- [ ] `get_contact_history` — Google Voice API / call log tracker
- [ ] `verify_drive_contents` — Google Drive API, file presence/type/size
- [ ] `dispatch_slack_handoff` — Slack Web API, transactional handoff
      to exactly 3 stakeholder accounts

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

### Dashboard (rep-facing)

- [ ] Prioritized queue view matching the system prompt's JSON output
      shape
- [ ] Missing-document checklist per lead
- [ ] Outreach template review/edit before send (not fully autonomous
      send in Phase 1 — confirm with Marc/Abdoul whether human-in-the-
      loop send approval is required at launch)

## Phase 1 — explicitly out of scope (confirm with owners before Phase 2 planning)

- Auto-send without rep review (pending decision — see decisions/)
- CRM integrations beyond Google Sheets
- Multi-tenant / multi-org support (single sales org for Phase 1)
- Historical analytics dashboard beyond the success metrics in the PRD
- Mobile app (dashboard is web-first)

## Notes

- Check items off only when built AND run against the eval card in
  testing/eval-suite.md — not just coded.
- The PRD (prd/LeadPilot_PRD_v1.md) is the authoritative source for
  full feature detail; this checklist tracks completion status only.
