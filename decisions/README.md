# decisions/

A log of every significant decision made during LeadPilot's build.
Each entry records what was decided, why, and what alternatives were
considered.

## What belongs here

- All decisions from the log below
- New decisions added during the build
- Decisions that were reversed or changed, with reason why

## Suggested file naming

  decisions-log.md      Single running log of all decisions

## Entry format

  ## Decision [number] — [short title]
  Date: [date]
  Decision: [what was decided]
  Reason: [why this was chosen]
  Alternatives considered: [what else was evaluated and why rejected]
  Status: Active / Superseded by Decision [number]

## Decisions log (current)

| # | Decision | Choice | Rationale |
|---|----------|--------|-----------|
| 001 | Agent name | LeadPilot | Reflects the "autonomous co-pilot for outbound sales" positioning |
| 002 | Ownership | Marc Delsoin, Abdoul Ba (co-owners) | Per PRD v1, July 6, 2026 |
| 003 | Docs repo structure | Adopted the NoiseToSignal docs schema, expanded (governance/, roadmap/, release-process/, monitoring-observability/, public/) | Reuse proven structure; add professional-repo elements identified as gaps before LeadPilot's build starts |
| 004 | Repo separation | Two private repos: `leadpilot` (code) and `leadpilot-docs` (planning/security/decisions) | Security separation and independent access control — see leadpilot-docs/README.md |
| 005 | Tech stack | Deferred | Not yet decided; expected to change substantially — see tech-stack/README.md |
| 006 | Prompt injection defense | Isolated validation layer strips instruction-like keywords from all spreadsheet-sourced text before tool execution | Per PRD 3c blast radius — indirect prompt injection is the identified worst-case failure mode |
| 007 | Duplicate contact prevention | Atomic state locking — run timestamps committed before tool calls are authorized | Per PRD 3c — timing-gap duplicate contact is a named failure mode with brand-damage impact |
| 008 | File completeness validation | Require file size >5KB and strict PDF extension match before counting a document as present | Per PRD 3c — prevents false-positive Slack handoffs on empty/invalid files |
| 009 | Outreach/handoff auto-send vs. rep approval | Nothing fires automatically — the agent only drafts (outreach, back-office handoff, spreadsheet edits); every side-effect action requires explicit rep approval via a single-use approval token | Resolves the Phase 1 open decision below. Marc/Abdoul decided the rep must stay in control of what actually sends, calls, or writes — see PRD v1.01 section 2 and the "changed to make" notes |
| 010 | Back-office handoff channel | The handoff can be either a phone call (`initiate_backoffice_call`) or a predetermined Slack message (`dispatch_slack_handoff`) to the same 3 stakeholders — both staged, both require rep approval | Per PRD v1.01 section 3b — expands the handoff beyond Slack-only in PRD v1 |
| 011 | Spreadsheet write-back | Added `update_lead_sheet` tool — reps edit lead data from the unified interface instead of the source sheets directly; writes only fire after the rep confirms a shown current-vs-proposed diff | Per PRD v1.01 1b/3a — removes the last reason a rep would need to leave the unified interface |
| 012 | Communications search | Added `search_communications` tool — searches a client's email/text history by any known identifier (email, phone, full name, company name), returning history, attachments, and confirmed documents | Per PRD v1.01 2a — a client is often reachable through more than one contact point; the rep needs history regardless of which one was used |
| 013 | Access control | Only authenticated, pre-authorized rep accounts may trigger an agent run, view lead/contact data, or approve a staged action | Per PRD v1.01 2c/3c — "lock down use of the agent by either the account using the agent or some other method" from the changed-to-make notes; resolved as an authenticated-session requirement |

## Open decisions (not yet made — track here as they resolve)

- Runtime/stack (tech-stack/README.md)
- Where dedup/run-lock state lives
- CI/CD and hosting choice
- Exact mechanism for the rep-approval token from Decision 009 (how
  it's minted, scoped, and expired — implementation detail, depends on
  stack choice)
- Whether `search_communications` (Decision 012) needs a dedicated
  compliance/retention review before use against real client data —
  see testing/known-issues-log.md Issue 004

## Notes

- Write new decisions at the time they are made, not weeks later.
- This log answers "why does it work this way?"
- When a decision is reversed, mark it Superseded and add the new
  decision with a reference back.
- Decisions 001-002 and 006-008 are sourced directly from
  prd/LeadPilot_PRD_v1.md; decisions 009-013 are sourced from
  prd/LeadPilot_PRD_v1.01.md and the raw notes in
  prd/LeadPilot_PRD_v1-changed to make.md.
