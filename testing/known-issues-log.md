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
Status: Partially resolved (2026-07-15) — calls place successfully;
SMS blocked by trial policy (upgrade required); metadata endpoints
still policy-blocked

**Update 2026-07-15 (later) — SMS root cause found via live probes.**
The only verified receiving number is +16467839391. With the correct
recipient the earlier 422/572002 (unverified recipient) changed to
**400/572006: "Invalid template name. Trial accounts can only use
predefined SMS templates"** — from both owned numbers, and even for
template-like plain text. This trial account is on Twilio's newer
restricted regime: custom SMS bodies cannot be sent at all.
Conclusion: `send_lead_text` (which sends rep-drafted content) cannot
work on this account until it's **upgraded off trial** — that is the
single unblock. Everything on LeadPilot's side (staging, gate,
approval, client, from-number, recipient verification) is proven
working right up to Twilio's paywall.

**Update 2026-07-15 — support's diagnostic script placed a real call.**
Marc ran `supporttest.py` and got a real call SID back
(`client.calls.create()` → 200), proving the account can place
outbound calls — whatever support adjusted (or the account state
change since) cleared the path that matters for actual telephony.
Immediately re-ran the read-only endpoint check
(`scripts/test_twilio_creds.sh`) the same day: the base Account
resource is still 200/active/Trial, but `IncomingPhoneNumbers` and
`OutgoingCallerIds` **still return `401 Policy evaluation failed`** —
so the original two endpoints remain blocked, and number-ownership /
verified-caller-ID checks still can't be done programmatically.
Practical consequences:
- `send_lead_text` (`messages.create`) very likely works now too —
  same family as the proven `calls.create` — but is still unverified
  live; trial-account rules still apply (recipients must be verified
  numbers, checkable only in the Twilio Console UI since the API
  endpoint is blocked).
- `TWILIO_FROM_NUMBER` ownership can't be API-confirmed; confirm in
  the Console. Note it's stored without a leading `+` in `.env.local`
  — Twilio generally wants E.164 (`+1...`); worth normalizing before
  the first live `send_lead_text`.
- Per Marc's own plan below: the diagnostic is done its job — delete
  `supporttest.py` and rotate the Account SID/Auth Token (the token
  sat in plaintext and was pasted into support tickets), then update
  `.env.local`.
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

## Issue 006 — Same-cell concurrent spreadsheet writes could silently clobber each other

Opened: 2026-07-13
Status: Resolved and fully verified (see Decision 034 in decisions/README.md) — the earlier verification gap below is closed
Description: Marc asked what happens if two reps (or two overlapping
runs) try to edit the same spreadsheet cell at once. Checked the real
code: `GoogleSheetsConnector.commit_field_write` was a plain
`values().update()` with no check against what the rep actually
approved — last-write-wins, no conflict detected, nothing logged.
`LeadActionLock`/`AgentRunLock` don't cover this (different concerns —
outreach cooldown and cron-run mutex, not spreadsheet cell writes).
Impact: A rep's approved edit could be silently overwritten by another
rep's (or the same rep's overlapping run's) approved edit to the same
cell, with no error, no log entry, and no way to tell it happened after
the fact.
Resolution: Decision 034 — `commit_field_write` now requires an
`expected_current` argument and raises `StaleWriteError` on mismatch;
a new `sheet_cell_locks` table serializes concurrent commits to the
same cell. Full mechanism, alternatives considered, and test coverage
in Decision 034's entry.

**Verified 2026-07-13, real evidence (Abdoul):** after `abdouls-branch`
and `marc-step2-split` were both merged into `main`, ran `alembic
upgrade head` against a real local Postgres — applied cleanly onto a
single unified head (`fed4e55c9f58`, chained correctly off
`cd645f125bf4`, confirming the "sibling migrations" prediction below
was right and no manual `alembic merge` was actually needed once both
branches landed in the same merge). Full suite: **182 passed**, 9
failed — every one of the 9 is `invalid_scope` on a live OAuth test,
unrelated to this fix (the locally stored refresh token predates
Decision 034's earlier Gmail-scope addition and needs a rep reconnect,
same reconnect-required situation Decision 033 already documented, not
a new problem).

**Migration-head question — resolved 2026-07-13, not actually a
mystery:** `cd645f125bf4` is real and confirmed — it's on
`origin/abdouls-branch` (tip `5bcae57` as of this correction, which
also carries all 5 of Group A's tools; see the mvp/README.md
correction below). It hasn't been fetched into `main`/
`marc-step2-split` yet, which is why it looked unreachable earlier.
`cd645f125bf4` and this fix's `fed4e55c9f58` are **sibling**
migrations — both branch off `d2caf87d6b35`, neither depends on the
other — so once `abdouls-branch` is merged into `main`, `alembic
heads` will correctly show two heads needing one ordinary `alembic
merge` to reconcile. Not a sign anything is broken, just the standard
two-people-added-a-migration-in-parallel situation.

**`update_lead_sheet` integration — done, not just planned (Abdoul,
2026-07-13, commit `3dc3a52` on `main`):** both predicted changes made,
exactly as anticipated below, plus one improvement beyond what was
flagged as a "UX nice-to-have":
1. `_encode_content_ref`/`_decode_content_ref` now carry a `current`
   field — `run()` persists `diff.current` at staging time, so
   `execute()` has it to pass as `expected_current`.
2. `execute()` passes `expected_current=info["current"]` — bracket
   access, not `.get()`, deliberately: this is a pre-launch dev system
   with no real production data, so there's no old-shape `content_ref`
   row that could lack the key and needs no defensive fallback for one.
3. **Beyond what was flagged as a nice-to-have:** rather than leaving
   `StaleWriteError`/`ConcurrentWriteError` wrapped inside the generic
   `WriteExecutionFailedAfterApprovalError` for Step 3 to unwrap later,
   `execute()` now re-raises both distinctly, unwrapped — Step 3's
   future approval endpoint can catch them directly without needing to
   inspect a wrapped exception's cause chain. Closes the precision gap
   this issue originally flagged as future work, now rather than later.

Also added `tests/test_update_lead_sheet.py::test_execute_raises_stale_write_error_if_the_cell_changed_since_staging`
— proves a stale approval is rejected rather than silently overwriting
another rep's (or a direct Sheets-UI) edit, and that the gate is still
correctly consumed (single-use survives a failed write, same documented
trade-off as any other write failure) even though the sheet itself was
never touched. `tests/fakes.py`'s `FakeLeadSourceConnector` was also
updated to enforce the same `expected_current`/`StaleWriteError`
contract the real connector does — it was stale prior to this fix and
had been silently masking the missing argument in every existing test.

## Issue 007 — No consent/opt-out/DNC suppression tracking for lead outreach

Opened: 2026-07-15
Status: Open
Description: Researching TCPA/CAN-SPAM/Do-Not-Call requirements
(Step 5 item, compliance/tcpa-can-spam-dnc-research.md) and checking
them against the real code found: no field anywhere (`Lead` model or
`contact_history`) records whether/when/how a lead consented to be
called/texted/emailed; no suppression list is checked before
`send_lead_text`, `send_lead_email`, or `initiate_lead_call` stage a
draft; no inbound Twilio webhook exists to capture a "STOP" reply into
LeadPilot's own state (grepped `src/leadpilot/*.py` and
`src/leadpilot/tools/*.py` directly — confirmed absent, not just
undocumented); `send_lead_email`'s drafted body has no unsubscribe
link, physical address, or advertisement disclosure. Separately, it's
unresolved whether contacted leads are individual consumers or
business entities — undecided, and it changes which rules apply (the
Do-Not-Call Registry's B2B exemption specifically).
Impact: A lead who opts out via a text reply, or verbally tells a rep
not to be contacted again, has no way to be suppressed from future
agent-drafted outreach — the agent could draft, and a rep could
unknowingly approve, another contact attempt. Full detail, proposed
(not built) fixes, and severity ranking in
`compliance/tcpa-can-spam-dnc-research.md`.

## Notes

- Move an issue to "Resolved" and reference the decisions/ entry that
  resolved it once fixed — don't delete the history.

## Issue 008 — .xlsx files are pickable-looking but unreadable (future version)

Opened: 2026-07-15 (Marc's first live Picker walkthrough)
Status: Open — deferred to a future version by Marc; picker-side
prevention shipped same day
Description: Google Drive treats uploaded `.xlsx` files as
"spreadsheets" for browsing purposes, but the Sheets API cannot read
them — only native Google Sheets. Marc picked four `.xlsx` files, all
four granted successfully, and every read then failed with the
connector's (correct) "granted but is not a Google Sheet" rejection.
Opening an `.xlsx` in Google Sheets does NOT convert it (Office
compatibility mode keeps the original file/mime); conversion requires
File → Save as Google Sheets, which creates a new file with a new id.
Mitigation shipped 2026-07-15: the sheets Picker view now filters to
`application/vnd.google-apps.spreadsheet` so unreadable files can't be
picked in the first place.
Future version: real `.xlsx` support (Drive download + openpyxl-style
parsing through the connector interface), or at minimum an in-UI
indicator/conversion prompt when a rep's Drive has `.xlsx` intake
files, so "my sheets don't show up" has an explanation in-product.
Impact: Reps whose intake data arrives as `.xlsx` (common from
exports) must manually convert each file before LeadPilot can read it.

## Issue 009 — Files in the Drive root aren't reachable via the folder-grant flow (future version)

Opened: 2026-07-15 (same walkthrough)
Status: Open — deferred to a future version by Marc
Description: The document-check flow grants Drive *folders*
(verify_drive_contents inspects a folder's contents). Files sitting
in the rep's My Drive root aren't inside any pickable folder, so they
can't be covered by a folder grant — the picker asks for a folder and
root-level files are invisible to the flow.
Future version: let the rep grant individual files for document
checks, and/or treat "root" as a grantable container.
Impact: Reps who keep deal documents loose in the Drive root get
"missing document" results even though the files exist.

## Issue 010 — Google Picker widget ignores the app's theme (future version)

Opened: 2026-07-15 (same walkthrough)
Status: Open — deferred; confirmed not quickly fixable
Description: The Picker renders white/light regardless of LeadPilot's
glass theme. Confirmed structural: the Picker is a Google-hosted
iframe with no theming/dark-mode API — nothing on our side can style
its internals. The only real fix is replacing the Picker with a
LeadPilot-native file browser, which is a significant lift because
Decision 026's `drive.file` scope model *requires* Google's own
Picker for per-file grants to register server-side (see the setAppId
lesson in the code repo's CLAUDE.md). A native browser would need a
scope-model rethink — pairs with Decision 033's flagged revisit.
Impact: Cosmetic — a bright white modal in an otherwise dark UI.

## Issue 011 — Status lives in cell background colors, not text (future version)

Opened: 2026-07-15 (Marc's live walkthrough)
Status: Open — needs a design discussion before building
Description: Marc's real intake sheets encode each lead's status as
the ROW'S BACKGROUND COLOR, with a legend row (above or below the
header row depending on the sheet) mapping colors to statuses (e.g. a
green swatch labeled FUNDED). The Sheets *values* API LeadPilot reads
returns text only — colors are completely invisible to it. Reading
them needs `spreadsheets.get` with `includeGridData=true` (a much
heavier response), color-matching each data row against the legend
swatches (with tolerance — theme colors vary), and resolving the
legend's color→status-name mapping from the legend row's cell fills +
text. Doable, but a real feature with real design questions (what
wins when a sheet has BOTH a Status column and row colors? what if a
row's color matches no swatch?).
Interim state shipped 2026-07-15: header-row auto-detection (legend
above headers no longer breaks ingestion), legend rows are skipped
instead of ingesting as junk "(no name)" leads, and the rep-confirmed
"create a Status column" flow gives status a text home LeadPilot can
read/write today. The v1 status vocabulary (Funded/Approved/Deal
In/App In/Interested/Dead, blank valid) is documented in
architecture/state-schema.md.
Impact: Until built, the status baked into existing sheets' colors is
invisible to LeadPilot — reps must populate the text Status column
(manually or via approved LeadPilot edits) for status to flow.

## Issue 012 — SSN/EIN/DOB from intake sheets stored and displayed verbatim

Opened: 2026-07-15 (spotted while mapping Marc's fuller intake sheets)
Status: Open — must be resolved as part of the Step 5 compliance/
security review, before real-lead launch
Description: Marc's fuller intake sheets carry `SSN`, `EIN`, and `DOB`
columns (plus home addresses). Ingestion stores every source column
verbatim in `lead_source_rows.raw_data` (JSONB, unencrypted — unlike
rep OAuth tokens, which are Fernet-encrypted per Decision 029), and
the interface's Source data panel displays the full row to any
authenticated rep. So the moment a sheet with an SSN column is
granted, LeadPilot is persisting and displaying government
identifiers with no special handling: no field-level encryption, no
masking, no access differentiation, no retention policy, and the
values also flow into whatever the agent loop reads
(`LeadSourceRow.raw_data` is documented as internal-only for
change-detection, but the Source data panel now surfaces it, and
raw rows reach model context via tool results during agent runs).
Questions to resolve (Step 5, alongside Issue 004's
search_communications review):
- Should PII-pattern columns (SSN/EIN/DOB) be masked in the UI by
  default (e.g. •••-••-1234 with click-to-reveal + audit log)?
- Should raw_data be field-level encrypted at rest, or PII columns
  dropped/tokenized at ingest?
- Should these columns be excluded from tool results the model sees
  during agent runs (the agent has no need for an SSN)?
- Retention: what happens to raw_data when a lead goes dead?
Impact: Compliance exposure (GLBA-adjacent for business-lending data;
state privacy laws for SSN handling) and breach blast radius. Doesn't
block local testing with sample data; blocks real-lead launch.
