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
Status: Open
Description: The PRD's `get_contact_history` tool assumes a
`GET /v1/communications/logs` endpoint exists for Google Voice. This
needs to be verified against actual Google Voice API documentation —
it's possible this requires a different integration approach (e.g. a
third-party call-tracking service instead of a native Google API).
Impact: Could change the architecture and tech stack choice if the
assumed API doesn't exist as described.

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
Status: Open
Description: PRD v1.01 requires a single-use, rep-approval token to
authorize any side-effect tool call, but the exact mechanism (how it's
minted, its scope/expiry, where it's validated) isn't defined yet —
this is an implementation detail that depends on the tech stack
decision.
Impact: Blocks a concrete architecture/dashboard-architecture.md and
architecture/state-schema.md until resolved.

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
