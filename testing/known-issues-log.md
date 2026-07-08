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
Status: Open
Description: The PRD implies the agent stages outreach templates and
handoffs, but doesn't explicitly confirm whether outreach is sent
automatically or requires rep review/approval first.
Impact: Affects mvp/README.md scope and the dashboard architecture.

## Notes

- Move an issue to "Resolved" and reference the decisions/ entry that
  resolved it once fixed — don't delete the history.
