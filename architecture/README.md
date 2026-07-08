# architecture/

Documents how LeadPilot works — the agent orchestration loop, tool
map, data flow between Google Workspace/Slack and the agent, and how
the rep-facing dashboard fits in.

## What belongs here

- System overview: agent runtime, tool layer, data stores, dashboard
- Data flow: step-by-step for the hourly run cycle
- Tool map: all seven tools, what each calls, what each returns (see
  prd/LeadPilot_PRD_v1.01.md section 3a for the source definitions —
  keep this file in sync when tools change)
- System prompt version history (copy of prd 3b plus any revisions)
- Database/state schema: how contact history, sync locks, and
  dedup state are stored
- Dashboard architecture: how the prioritized queue reaches the rep

## Suggested file naming

  system-overview.md       High-level architecture
  data-flow.md             Step-by-step hourly run cycle
  tool-map.md              All tools, inputs, outputs, APIs called
  system-prompt-history.md Versioned copy of the system prompt
  state-schema.md          Contact history / dedup / lock state storage
  dashboard-architecture.md How the rep-facing queue view is served

## Current architecture (Phase 1 — draft, tech stack undecided)

```
Hourly scheduler
      |
      v
LeadPilot agent runtime
      |
      |-- fetch_all_leads ----------> Google Sheets API
      |-- get_contact_history ------> Google Voice API / call log tracker
      |-- verify_drive_contents ----> Google Drive API
      |-- search_communications ----> Email/SMS provider search APIs (read-only)
      |
      v
Prioritized queue + recommended actions (JSON) --> state store (dedup + run-lock)
      |
      v
Rep-facing unified interface (review queue, edit/approve outreach,
inline spreadsheet editing with diff preview, communications search)
      |
      | -- rep clicks "Approve" --> mints single-use approval token
      v
      |-- dispatch_slack_handoff ----> Slack Web API   (staged, fires only w/ valid token)
      |-- initiate_backoffice_call --> Google Voice API (staged, fires only w/ valid token)
      |-- update_lead_sheet ---------> Google Sheets API (staged, fires only w/ valid token)
```

## Key architecture notes

- The agent's read/prioritization sequence is fixed in the system
  prompt (v1.01): fetch leads -> cross-reference contact history ->
  prioritize -> verify drive contents -> draft recommended actions ->
  draft back-office handoff if complete. Any change to this sequence
  needs a decisions/ entry, since the eval card
  (testing/eval-suite.md) is written against this exact order.
- **Nothing with a real-world side effect executes as part of the
  agent's own run.** `dispatch_slack_handoff`, `initiate_backoffice_call`,
  and `update_lead_sheet` all stage a draft; the actual API call only
  fires when the interface presents a valid, single-use rep-approval
  token minted at the moment the rep clicks approve (Decision 009 in
  decisions/README.md). This is the architecture's most important
  invariant post-v1.01 — treat any code path that could fire one of
  these tools without a token as a security bug, not a convenience
  shortcut.
- Atomic state locking (decisions pending on exact implementation —
  file-based, Postgres, Redis, etc.) must commit run timestamps
  *before* tool calls are authorized, to prevent duplicate contact
  (see security/threat-model.md, failure mode: duplicate contact
  tracking failure).
- Every agent run, data view, and approval requires a valid
  authenticated rep session (Decision 013) — this gate sits in front
  of the entire diagram above, not just the approval step.

## Tech stack status

Runtime/language is not yet decided (Python agent loop vs Node.js vs
other) — see tech-stack/README.md. This file will be updated with
concrete component names once that's locked.

## Notes

- Update this folder whenever the architecture changes.
- Outdated docs are worse than no docs.
- Write for your future self and for Abdoul, who may pick this up
  without you in the room.
