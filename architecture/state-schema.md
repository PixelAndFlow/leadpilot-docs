# State schema — contact history, dedup, and run-lock state

Resolves the `get_contact_history` half of `testing/known-issues-log.md`
Issue 001: since Google Voice has no API to poll, LeadPilot maintains
its own contact-history log instead of querying any external call-log
service. `get_contact_history` reads from this log, not from Google
Voice — see decisions/README.md Decision 018.

## Contact history log

One append-only record per contact-related event. Every tool that
touches a lead's contact history writes an entry here at the moment
its status changes — this is what `get_contact_history` reads, and
what the prioritization logic (PRD system prompt step 3) uses to
determine Rank 1/2/3.

| Field | Type | Notes |
|---|---|---|
| `event_id` | string (uuid) | Unique per log entry |
| `lead_id` | string | Canonical lead identifier — must survive de-dup merges (Eval Case 2), so this is resolved *after* dedup, not the raw per-sheet row id |
| `channel` | enum | `call` / `text` / `email` / `slack_handoff` / `sheet_edit` |
| `tool` | string | Which tool produced this event: `initiate_lead_call`, `send_lead_text`, `send_lead_email`, `dispatch_slack_handoff`, `update_lead_sheet` |
| `stage` | enum | `drafted` -> `awaiting_rep_approval` -> `approved` -> `executed` / `rejected` / `expired` |
| `timestamp` | datetime | When this stage transition happened |
| `rep_id` | string | Which authenticated rep triggered the transition (null for `drafted`) |
| `outcome` | enum, nullable | See "Outcome visibility" below — this is where the gap is |
| `content_ref` | string | Reference/snippet of the drafted content (subject to the retention policy still open in compliance/README.md) |
| `note` | string, nullable | Free text — primarily used for rep-reported call outcomes |

## Rep-approval mechanism (resolves Issue 003)

The `stage` field in the contact-history log above isn't just a record
of what happened — it's the actual enforcement mechanism for the
rep-approval gate. There's no separate "staged actions" table and no
signed or opaque token object. A staged action's row moves through:

`drafted -> awaiting_rep_approval -> approved -> executed` (or
`rejected` / `expired`)

The rep's "Approve" click is an authenticated request that flips one
specific row from `awaiting_rep_approval` to `approved`. Execution
then runs a single conditional update:

```sql
UPDATE contact_history
SET stage = 'executed'
WHERE event_id = ? AND stage = 'approved'
```

If that update affects exactly one row, the tool is cleared to run its
real effect (send the text, post the Slack message, write the sheet,
copy the number to the clipboard). If it affects zero rows — because
the action was never approved, was already executed, or was
rejected/expired — nothing runs. That single atomic conditional update
is what makes "single-use" true: there's no window where two
concurrent requests could both see `approved` and both fire, since the
database only lets one of them win the row.

This needs no cryptography and no separate token artifact. Weighed
against a signed JWT (unnecessary — the token never leaves the
backend, so signing buys nothing a database check doesn't already
give you) and an opaque token in its own table (functionally
redundant — it would just duplicate this same state machine), the
status-flip approach was chosen as the simplest correct mechanism —
see Decision 021 in decisions/README.md. It pairs with the
authenticated-session requirement (Decision 013): the session check
confirms *who* is clicking approve; this state machine confirms *that
specific action* fires exactly once.

**What this needs from the eventual tech stack:** whichever database
is chosen (tech-stack/README.md, still open) needs to support this
kind of atomic conditional update (`UPDATE ... WHERE stage = X`,
checking affected-row-count) — standard in SQLite and Postgres alike,
so it shouldn't constrain that choice. It should be tested under
concurrent access once the stack is picked, to confirm only one
request ever wins the row — a concurrency test, not just the
sequential eval-card cases (5, 7, 9), which check the gate's behavior
but not a true race.

## Outcome visibility — the gap this design doesn't fully close

For `send_lead_text`, `send_lead_email`, and `dispatch_slack_handoff`,
LeadPilot is the one actually sending, so it gets a real delivery
confirmation from the provider API and can log a real `outcome`
(`delivered` / `failed`).

For `initiate_lead_call`, that's not true anymore. Since PRD v1.03,
approval only copies the number to the rep's clipboard — the rep
places the call entirely outside LeadPilot, in their own calling tool.
LeadPilot can log that the call was *approved and the number was
handed off* (`stage: executed`, `outcome: pending`), but it has no way
to automatically know whether the call was answered, went to
voicemail, or wasn't placed at all.

This matters beyond record-keeping: the system prompt's Rank 3 logic
("if a call went unanswered, stage an explicit Text or Email
follow-up") depends on knowing the call's outcome. Without it, that
cadence rule has no data to run on for calls specifically.

**Fix (built into PRD v1.04):** `log_call_outcome` is a rep-facing
tool — a quick prompt or button next to a just-approved call
(`Answered` / `No answer` / `Voicemail` / `Didn't call`) that writes
the `outcome` and `note` fields onto that call's existing log entry
after the fact. It's rep-initiated (the rep is reporting a fact about
a call they placed themselves), writes only to this internal log, and
doesn't need the rep-approval-token gate that the actual outreach/
handoff tools require — there's no external system on the other side
of it. Until the rep reports an outcome, the entry stays
`outcome: pending` and Rank 3's unanswered-call follow-up rule has
nothing to act on for that lead (PRD v1.04 Eval Case 10 covers both
the positive and negative case). Texts, emails, and Slack handoffs are
unaffected since their outcomes come back automatically from their
provider APIs.

## Storage

Not tied to a specific database — tech stack is still undecided
(tech-stack/README.md). Recommended starting point: an append-only
JSONL file (one JSON object per line, one file or one per lead),
since it requires no stack decision to start with, is human-readable,
and diffs cleanly. Migrate to a real table (SQLite now, Postgres if/when
hosted) once the tech stack is locked — the schema above maps directly
to a table with the same columns; `lead_id` and `timestamp` should be
indexed once it does.

## Dedup / run-lock state

Designed in Decision 025 as three tables (`lead_source_rows`,
`lead_action_locks`, `agent_run_locks`) — see `leadpilot/models/dedup.py`,
`leadpilot/models/run_lock.py`, and `leadpilot/locks.py` in the code
repo, built and tested in Step 1.

**Needs revisiting (Decision 027):** `agent_run_locks` was designed as
a singleton mutex — one row, so the hourly Cron Job can't run twice
concurrently. Now that the batch run is per-rep rather than global
(Decision 027), this needs to become a per-rep mutex (e.g. keyed by
`rep_id`, so rep A's run and rep B's run can proceed independently, but
two concurrent runs for the *same* rep still can't overlap) instead of
one lock for the whole job. Exact schema change is Step 2 work, not
designed here yet — flagging so it isn't missed when `fetch_all_leads`
is actually built against the per-rep model.

## Rep Google credentials (new — Decision 026)

Needed once Sheets/Drive access moves to per-rep OAuth. Not yet built
— flagging the shape so Step 2 has something concrete to start from,
not a full schema design:

| Field | Type | Notes |
|---|---|---|
| `rep_id` | string | FK to the rep table (`leadpilot/models/rep.py`) |
| `refresh_token` | string, encrypted at rest | The rep's Google OAuth refresh token (`drive.file` scope) — used for both on-demand `fetch_ad_hoc_sheet` calls and that rep's hourly batch run |
| `granted_file_ids` | list of strings | Sheet/Drive file IDs the rep has selected via the Google Picker — this is what `list_sources()` (3e) enumerates for that rep |
| `connected_at` | datetime | When the rep completed the OAuth consent |
| `revoked_at` | datetime, nullable | Set if the rep disconnects their Google account or revokes access |

Encryption-at-rest approach for `refresh_token` (KMS-backed column
encryption vs. application-level encryption before insert) is not yet
decided — needs to land alongside the broader secrets-management
open item (decisions/README.md) before Step 2 builds this table.
