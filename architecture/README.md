# architecture/

Documents how LeadPilot works — the agent orchestration loop, tool
map, data flow between Google Workspace/Slack and the agent, and how
the rep-facing dashboard fits in.

## What belongs here

- System overview: agent runtime, tool layer, data stores, dashboard
- Data flow: step-by-step for the hourly run cycle
- Tool map: all ten tools, what each calls, what each returns (see
  prd/LeadPilot_PRD_v1.04.md section 3a for the source definitions —
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
      |-- fetch_all_leads ----------> LeadSourceConnector --> Google Sheets API
      |                                (GoogleSheetsConnector now;
      |                                 Excel/OnlyOffice/LibreOffice/
      |                                 OpenOffice/Docs planned later,
      |                                 see PRD v1.02 section 3e)
      |-- get_contact_history ------> LeadPilot's own contact-history log
      |                                (state store — see architecture/
      |                                 state-schema.md; no external API)
      |-- verify_drive_contents ----> Google Drive API
      |-- search_communications ----> Email/SMS provider search APIs (read-only)
      |
      v
Prioritized queue + recommended actions (JSON) --> state store (contact
                                                     history + dedup + run-lock)
      |
      v
Rep-facing unified interface (review queue, edit/approve outreach,
inline spreadsheet editing with diff preview, communications search,
report a call's outcome)
      |
      | -- rep clicks "Approve" --> mints single-use approval token
      v
      |-- initiate_lead_call --------> Clipboard write only, no API (staged, fires only w/ valid token)
      |-- send_lead_text ------------> SMS provider API  (staged, fires only w/ valid token)
      |-- send_lead_email -----------> Email provider API (staged, fires only w/ valid token)
      |-- dispatch_slack_handoff ----> Slack Web API — completion handoff, info request, or
      |                                 urgent_callback_request (staged, fires only w/ valid token;
      |                                 replaces the retired initiate_backoffice_call)
      |-- update_lead_sheet ---------> LeadSourceConnector --> Google Sheets API (staged, fires only w/ valid token)

      | -- rep reports call outcome (no token needed, rep-sourced fact) --
      v
      |-- log_call_outcome ----------> writes directly to the contact-history log (state store)
```

## Key architecture notes

- The agent's read/prioritization sequence is fixed in the system
  prompt (v1.04): fetch leads -> cross-reference contact history ->
  prioritize -> verify drive contents -> draft recommended actions ->
  draft back-office handoff if complete. Any change to this sequence
  needs a decisions/ entry, since the eval card
  (testing/eval-suite.md) is written against this exact order.
- **Nothing with a real-world side effect executes as part of the
  agent's own run.** `dispatch_slack_handoff` (every handoff type,
  including `urgent_callback_request`), `update_lead_sheet`,
  `initiate_lead_call`, `send_lead_text`, and `send_lead_email` all
  stage a draft; the actual effect only fires once that action's
  contact-history log row is flipped to `approved` and a single atomic
  conditional update confirms it hasn't already executed (Decision 009
  in decisions/README.md, extended to the outreach tools by Decision
  014, extended again to cover urgent back-office handoffs by Decision
  019 — urgency is never a reason to skip the gate, mechanism finalized
  by Decision 021 — see architecture/state-schema.md; no separate
  token object exists). This is the architecture's most important
  invariant post-v1.01 — treat any code path that could fire one of
  these tools without that row being in `approved` state as a security
  bug, not a convenience shortcut. For `initiate_lead_call`
  specifically, that "effect" is a clipboard write and a confirmation
  message, not a network call — see the next bullet. `log_call_outcome`
  is deliberately *not* part of this gate (Decision 020) — it's the rep
  reporting a fact about a call they already placed, writes only to
  the internal contact-history log, and has no external effect to gate.
- **`initiate_lead_call` never touches Google Voice or any calling app
  (Decision 016 in decisions/README.md).** Google Voice has no official
  API, and its Acceptable Use Policy explicitly prohibits scripting or
  automating call placement — even a script that only pre-fills the
  dialer and stops. So rep approval copies the lead's phone number to
  the clipboard and shows a confirmation message; the rep dials
  manually in their own calling tool.
- **Google Voice dependency is fully retired (Issue 001, resolved).**
  `get_contact_history` reads LeadPilot's own contact-history log
  (Decision 018, architecture/state-schema.md) instead of any external
  call-log API, and `initiate_backoffice_call` is retired in favor of
  `dispatch_slack_handoff`'s `urgent_callback_request` type
  (Decision 019). No tool anywhere in this architecture depends on
  Google Voice having an API it doesn't have.
- **Lead data access goes through a `LeadSourceConnector` interface**
  (`list_sources`, `fetch_rows`, `write_field`, `detect_changes`),
  not a direct Sheets API call from business logic (Decision 015 in
  decisions/README.md). Only `GoogleSheetsConnector` exists today.
  Excel and OnlyOffice are expected to be straightforward
  implementations of the same interface later; LibreOffice/OpenOffice
  will need a polling-based `detect_changes` instead of a webhook, and
  a Google Docs connector needs its own lead-record data-modeling
  decision before it can be built at all (see PRD v1.02 section 3e and
  the open items in decisions/README.md).
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
