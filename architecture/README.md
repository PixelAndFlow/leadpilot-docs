# architecture/

Documents how LeadPilot works — the agent orchestration loop, tool
map, data flow between Google Workspace/Slack and the agent, and how
the rep-facing dashboard fits in.

## What belongs here

- System overview: agent runtime, tool layer, data stores, dashboard
- Data flow: step-by-step for the hourly run cycle
- Tool map: all four tools, what each calls, what each returns (see
  prd/LeadPilot_PRD_v1.md section 3a for the source definitions —
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
      |-- fetch_all_leads --------> Google Sheets API
      |-- get_contact_history ----> Google Voice API / call log tracker
      |-- verify_drive_contents --> Google Drive API
      |-- dispatch_slack_handoff -> Slack Web API
      |
      v
Prioritized queue (JSON) --> state store (dedup + run-lock)
      |
      v
Rep-facing dashboard (review queue, edit/approve outreach, advance file)
```

## Key architecture notes

- The agent's tool-call sequence is fixed in the system prompt (v0):
  fetch leads -> cross-reference contact history -> prioritize ->
  verify drive contents -> dispatch handoff if complete. Any change to
  this sequence needs a decisions/ entry, since the eval card
  (testing/eval-suite.md) is written against this exact order.
- Atomic state locking (decisions pending on exact implementation —
  file-based, Postgres, Redis, etc.) must commit run timestamps
  *before* tool calls are authorized, to prevent duplicate contact
  (see security/threat-model.md, failure mode: duplicate contact
  tracking failure).
- Whether outreach is auto-sent or requires rep approval before send
  is an open decision — see decisions/decisions-log.md and
  mvp/README.md.

## Tech stack status

Runtime/language is not yet decided (Python agent loop vs Node.js vs
other) — see tech-stack/README.md. This file will be updated with
concrete component names once that's locked.

## Notes

- Update this folder whenever the architecture changes.
- Outdated docs are worse than no docs.
- Write for your future self and for Abdoul, who may pick this up
  without you in the room.
