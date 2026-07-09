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
| 014 | Lead outreach execution model | Added `initiate_lead_call`, `send_lead_text`, `send_lead_email` — all stage drafts only, same rep-approval gate as every other side-effect tool, no autonomous send | Closes a gap in v1.01: lead outreach (call/text/email) was required by 1b, section 2, the system prompt, and Eval Case 1, but no tool existed to back it. Marc confirmed outreach follows the same approval gate as everything else — no carve-out — see PRD v1.02 section 3a |
| 015 | Lead-source connector architecture | Introduced a `LeadSourceConnector` interface (`list_sources`, `fetch_rows`, `write_field`, `detect_changes`); only `GoogleSheetsConnector` is built now, behind `fetch_all_leads`/`update_lead_sheet` | Keeps prioritization/next-best-action logic from hard-coding Sheets-specific assumptions, so later sources don't require reworking the tools that consume them — see PRD v1.02 section 3e |
| 016 | Lead call handoff mechanism | `initiate_lead_call` no longer calls any telephony API. On rep approval, LeadPilot copies the lead's phone number to the clipboard and shows a confirmation message; the rep places the call manually in whatever tool they use | Google Voice has no official public API (confirmed), and its Acceptable Use Policy explicitly prohibits scripting or automating the placement of calls/messages, including stopping short of the final dial — so pre-filling Voice's dialer via browser automation is still a policy violation, not just a fragility risk. Clipboard handoff never touches Voice or any calling app, so neither problem applies — see PRD v1.03 section 3a and `testing/known-issues-log.md` Issue 001 |
| 017 | Equity split and decision rights | Equal 50/50 ownership between Marc and Abdoul; either partner may fork or use the work as they see fit | Confirmed directly by Marc, 2026-07-08 — documented in governance/README.md. Still flagged there as needing a real written agreement, since "use it as they see fit" doesn't on its own resolve naming/data/liability questions if one partner forks solo |
| 018 | Contact history data source | `get_contact_history` reads from a LeadPilot-owned, append-only contact-history log instead of any Google Voice API | Google Voice has no API to poll (Issue 001); LeadPilot already knows about every action it stages/sends, so it can log its own history rather than depending on an external call-log service. See architecture/state-schema.md and PRD v1.04 section 3f. Call-outcome gap closed by Decision 020 (`log_call_outcome`) |
| 019 | Back-office handoff channel (supersedes Decision 010) | `initiate_backoffice_call` is retired. Back-office handoffs are always a Slack message via `dispatch_slack_handoff`, including a new `urgent_callback_request` message type for what used to be the call option — still rep-approved, no autonomous-send exception | Closes Issue 001 entirely — no tool depends on a Google Voice API anymore. Confirmed directly with Marc that urgency is not a reason to skip rep approval, even for an internal-only recipient — see PRD v1.04 section 3a/3b and Eval Case 9 |
| 020 | Call-outcome logging | Added `log_call_outcome` — a rep-initiated tool (not agent-drafted, no approval token needed) that records answered/no-answer/voicemail/didn't-call against a call's contact-history log entry | Closes the gap flagged in Decision 018: since calls happen entirely outside LeadPilot (Decision 016), outcomes aren't automatically knowable. Without this, Rank 3's unanswered-call follow-up rule has no data for calls specifically. See PRD v1.04 section 3a/3f and Eval Case 10 |
| 021 | Rep-approval enforcement mechanism (resolves Issue 003) | No separate token object. The contact-history log row (Decision 018) carries a `stage` field (`drafted -> awaiting_rep_approval -> approved -> executed`/`rejected`/`expired`); approval flips it to `approved`, and execution does a single atomic conditional update (`UPDATE ... SET stage='executed' WHERE stage='approved'`), checking exactly one row was affected before firing the real effect | Simplest correct mechanism — the same row that's already the audit trail (Decision 018) is also the enforcement point, so there's no second system to build or keep in sync. Pairs with the authenticated-session requirement (Decision 013): session confirms who approved, this confirms the action fires exactly once. See architecture/state-schema.md. Alternatives considered: a signed JWT-style token (rejected — the token never leaves the backend, so cryptographic signing adds complexity without adding real security over a database check); an opaque token in a separate table (rejected — functionally redundant with the state machine, would duplicate bookkeeping) |

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
- Data-modeling decision for a `GoogleDocsConnector` (Decision 015) —
  what structure counts as "one lead record" inside a free-form Doc
  (Docs-native table vs. consistent heading-per-lead vs. something
  else) needs to be settled before that connector can even be
  estimated, let alone built
- Build order/timeline for the remaining planned connectors (Excel via
  Microsoft Graph, OnlyOffice Docs Server, LibreOffice/OpenOffice) —
  none scheduled yet, no validated need beyond Google Sheets so far
- Exact UI placement/flow for `log_call_outcome` (Decision 020) — the
  tool exists in the PRD; the interface design for prompting the rep
  at the right moment isn't done yet
- Concurrency test for the Decision 021 conditional update — needs to
  be written once the tech stack (and therefore the actual database)
  is chosen, to confirm the atomic update genuinely prevents a double
  execution under real concurrent load, not just sequentially

## Notes

- Write new decisions at the time they are made, not weeks later.
- This log answers "why does it work this way?"
- When a decision is reversed, mark it Superseded and add the new
  decision with a reference back.
- Decisions 001-002 and 006-008 are sourced directly from
  prd/LeadPilot_PRD_v1.md; decisions 009-013 are sourced from
  prd/LeadPilot_PRD_v1.01.md and the raw notes in
  prd/LeadPilot_PRD_v1-changed to make.md; decisions 014-015 are
  sourced from prd/LeadPilot_PRD_v1.02.md; decision 016 is sourced
  from prd/LeadPilot_PRD_v1.03.md; decisions 017-018 were made in
  direct conversation with Marc (2026-07-08) and are now reflected in
  prd/LeadPilot_PRD_v1.04.md; decisions 019-020 are sourced from
  prd/LeadPilot_PRD_v1.04.md; decision 021 is an architecture-level
  decision (not a PRD change) — see architecture/state-schema.md.
