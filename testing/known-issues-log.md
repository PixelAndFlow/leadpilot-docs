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

## Issue 005 — Twilio trial account blocks phone-number API endpoints

Opened: 2026-07-12
Status: Open
Description: Marc's Twilio trial credentials (`TWILIO_ACCOUNT_SID`/
`TWILIO_AUTH_TOKEN`) authenticate fine against the base Account
resource (`GET /Accounts/{SID}.json` → 200, account status `active`,
type `Trial`) — this resolves the plain 401 flagged in Decision 032's
commit message. But the same credentials return `401 Policy
evaluation failed` against `IncomingPhoneNumbers` and
`OutgoingCallerIds`, the two endpoints needed to confirm
`TWILIO_FROM_NUMBER` is actually owned by the account and to check
which recipient numbers are verified (trial accounts can only send to
verified numbers). Reproduced identically on two unrelated networks,
which rules out a local firewall/DLP/VPN intercepting the request —
this points at something on Twilio's account side, not a network or
credentials-formatting problem. Root cause not yet confirmed; "Policy
evaluation failed" doesn't match any documented Twilio error code, so
this needs either the Twilio Console UI checked directly for an
account-restriction/verification banner, or Twilio support contacted
with the exact error string.
Impact: **Updated 2026-07-13 — all 11 Step 2 tools are now built,
including `send_lead_text` and `search_communications`.** This issue
no longer blocks *writing* code, only *live-verifying* it: both
tools' Twilio-facing execute paths use an injectable client
(`twilio_client=` parameter) so their staging/gating/send logic is
unit-tested against a fake client, not the real API. `messages.create()`
(send_lead_text) and `messages.list()` (search_communications) are
different Twilio endpoints from the two that are actually failing
(`IncomingPhoneNumbers`, `OutgoingCallerIds`) — they may well work
fine even with this issue unresolved, but that's genuinely unverified
either way until someone runs them against a real Twilio account.
Treat both tools as code-complete and tested-against-fakes, not
confirmed-working, until this issue resolves or someone runs them live.

**Update 2026-07-13 — Twilio support provided a diagnostic script.**
As part of investigating the `Policy evaluation failed` error above,
Twilio support gave Marc a standalone test script to place a single
outbound test call directly against `client.calls.create()`,
independent of LeadPilot's own code. This is diagnostic tooling for
this issue, not part of the product — it's not one of the 11 Step 2
tools and isn't wired into anything the agent calls.

Two files came out of this: `leadpilot/scripts/supporttest.py` (the
working copy Marc filled in with his real `TWILIO_ACCOUNT_SID`/
`TWILIO_AUTH_TOKEN` in plain text — support's template expects them
typed in directly, not read from `.env.local`) and
`leadpilot/scripts/twilio_call_diagnostic_template.py` (renamed from
support's original `support.orig`, placeholder credentials only).

- `supporttest.py` is listed by name in `leadpilot/.gitignore`
  (`supporttest.py`, with an inline comment explaining why) so it can
  never be committed with real credentials in it — confirmed via
  `git status`/`git check-ignore` that it's untracked and actively
  ignored, not merely absent from a stray `git add`. **Per Marc's
  plan, this file gets deleted and the Twilio Account SID/Auth Token
  regenerated once this diagnostic testing is complete** — the
  credentials typed into it should be treated as exposed (even though
  never pushed) and rotated on that basis, not reused going forward.
- `twilio_call_diagnostic_template.py` is **kept intentionally**, not
  deleted — it's a reusable troubleshooting tool for the next time
  something in the Twilio integration misbehaves (isolates "LeadPilot
  bug" from "Twilio account issue" the same way it did for this
  issue). Committed to the repo since it only ever holds placeholder
  credentials; has a header comment marking it as diagnostic-only, not
  needed for the build, and meant to be copied/filled-in/discarded
  per-use rather than edited in place.

Move this issue to Resolved once `supporttest.py` is deleted, the
Twilio credentials are rotated, and the root cause of `Policy
evaluation failed` is confirmed.

## Notes

- Move an issue to "Resolved" and reference the decisions/ entry that
  resolved it once fixed — don't delete the history.
