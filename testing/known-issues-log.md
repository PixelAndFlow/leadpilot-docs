# Known issues log

Open bugs, gaps, and unresolved questions — distinct from
decisions/decisions-log.md, which records decisions already made.
This log records things that are still broken, unverified, or
unknown.

## Format

  ## Issue [number] — [short title]
  Opened: [date]
  Status: Open / In progress / Resolved (see Decision [#] if resolved)
  Description: [what's wrong or unknown]
  Impact: [who/what this affects]

## Current known issues

## Issue 001 — Google Voice API access model unverified

Opened: 2026-07-07
Status: Resolved (see Decisions 016, 018, 019, 020 in decisions/README.md)
Description: The PRD's `get_contact_history` tool assumes a
`GET /v1/communications/logs` endpoint exists for Google Voice. Since
opening this issue, we confirmed directly: Google Voice has no
official public API for calls or messages, and its Acceptable Use
Policy explicitly prohibits scripting or automating call/message
placement (temporary block escalating to account suspension for
violations).
Impact: Could change the architecture and tech stack choice if the
assumed API doesn't exist as described.
Resolution: No tool in LeadPilot depends on a Google Voice API anymore.
`initiate_lead_call` (PRD v1.03) copies the lead's phone number to the
clipboard on rep approval instead of calling any telephony API — the
rep dials manually. `get_contact_history` (PRD v1.04) reads from a
LeadPilot-owned contact-history log instead of any external call-log
service (architecture/state-schema.md, Decision 018).
`initiate_backoffice_call` is retired (PRD v1.04, Decision 019) — the
back-office handoff, including what used to be the call option, is
always a Slack message via `dispatch_slack_handoff` now. The
call-outcome visibility gap (LeadPilot can't automatically know if a
call was answered) is closed by `log_call_outcome` (PRD v1.04,
Decision 020), a rep-initiated report, not an automatic signal.

## Issue 002 — Auto-send vs. rep-approval workflow undecided

Opened: 2026-07-07
Status: Resolved (see Decision 009 in decisions/README.md)
Description: The PRD implies the agent stages outreach templates and
handoffs, but doesn't explicitly confirm whether outreach is sent
automatically or requires rep review/approval first.
Impact: Affects mvp/README.md scope and the dashboard architecture.
Resolution: PRD v1.01 decided nothing fires automatically — every
side-effect action (outreach, back-office handoff, spreadsheet write)
requires explicit rep approval via a single-use approval token. See
prd/LeadPilot_PRD_v1.01.md section 3a ("Execution-gating rule").

## Issue 003 — Rep-approval token mechanism not yet specified

Opened: 2026-07-07
Status: Resolved (see Decision 021 in decisions/README.md)
Description: PRD v1.01 requires a single-use, rep-approval token to
authorize any side-effect tool call, but the exact mechanism (how it's
minted, its scope/expiry, where it's validated) isn't defined yet —
this is an implementation detail that depends on the tech stack
decision.
Impact: Blocks a concrete architecture/dashboard-architecture.md and
architecture/state-schema.md until resolved.
Resolution: No separate token object. The contact-history log row
(Decision 018) carries the approval state directly via a `stage`
field; approval flips it to `approved`, and a single atomic
conditional update (`UPDATE ... WHERE stage='approved'`, checking
exactly one row affected) gates the real effect — see
architecture/state-schema.md. The concurrency test this issue flagged
as still open is now written and passing:
`leadpilot/tests/test_gate.py::test_try_execute_is_single_use_under_concurrency`
fires 10 real simultaneous execute attempts at the same approved row
against real Postgres — exactly one wins, every time (Abdoul,
2026-07-10, Step 1).

## Issue 004 — Communications-search compliance review needed

Opened: 2026-07-07
Status: Open
Description: The new `search_communications` tool (PRD v1.01) reads
actual message content and attachments across email and text, not
just contact metadata like the existing tools. This is a larger
data-exposure surface and hasn't had a compliance/retention review.
Impact: Should be resolved before this tool is used against real
client data — see compliance/README.md.

## Notes

- Move an issue to "Resolved" and reference the decisions/ entry that
  resolved it once fixed — don't delete the history.
