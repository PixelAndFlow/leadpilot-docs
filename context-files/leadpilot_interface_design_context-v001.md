# LeadPilot — Interface Design Context v001

Paste this at the start of a session with a design-focused LLM to get
help designing/wireframing LeadPilot's Step 3 rep-facing dashboard.
Distinct from `leadpilot_context-v002.md` (that one restores general
project state; this one is scoped tightly to what an interface
designer needs: screens, real data shapes, and hard interaction
constraints). If asked for product/security/decision context beyond
what's here, pull from `leadpilot_context-v002.md` instead — it's
overdue for a refresh (still says "7 tools," current build has 11) so
treat anything outside this file's scope as unverified until checked
against the live docs repo.

## What this is, in one paragraph

LeadPilot is an AI sales agent (Claude Agent SDK tool-calling loop)
for outbound sales reps. It reads leads from Google Sheets, checks
contact history to avoid double-outreach, verifies required documents
are in Google Drive, and drafts next actions (call/text/email,
back-office Slack handoff, spreadsheet edits) — but drafts only. This
interface is the *only* place any of those drafts becomes real. Every
side-effect action in the product routes through this dashboard's
approve/reject flow; there is no other path to "send."

## Current build status (as of 2026-07-13)

Backend is done: all 11 agent tools exist, tested, split between two
developers (Marc, Abdoul). The approval-gate state machine and auth
system are built and tested against real Postgres. **None of the
interface described below exists yet** — this file is context for
designing it from scratch. Tech stack is locked (see below), not up
for debate in this design pass.

## Non-negotiable interaction pattern — design around this, don't fight it

Every side-effecting row in this product moves through one state
machine:

```
drafted -> awaiting_rep_approval -> approved -> executed
                                  -> rejected
                                  -> expired
```

The UI's entire job for anything staged is: show the draft, let the
rep approve or reject it, and never let anything fire without that
explicit click. This applies identically to every action type,
**including urgent back-office callback requests** — urgency changes
how something should look (more visually prominent, maybe sorted to
the top), never whether it needs approval. If a design ever proposes
an "auto-approve" or "send immediately for urgent items" shortcut,
that's a hard no — it contradicts the product's core value
proposition, not a style preference.

One exception: `log_call_outcome` (the rep reporting what happened on
a call they already placed) and read-only lookups
(`get_contact_history`, `search_communications`, `fetch_all_leads`,
`verify_drive_contents`, `fetch_ad_hoc_sheet`) need no approval step —
there's nothing external to gate.

## Tech constraints that shape what's actually feasible

- **Server-rendered Jinja2 templates + htmx, not a React/Vue SPA.**
  Chosen deliberately (small two-person team, no separate JS build
  pipeline) — design within what htmx does well: partial page
  swaps/updates triggered by clicks, not client-side state management
  or complex animated transitions. If a design needs heavy client-side
  interactivity beyond that, flag it as a scope question, don't just
  assume it's buildable as-is.
- **One vanilla JS exception:** `navigator.clipboard.writeText()` for
  the `initiate_lead_call` approval flow (see below) — this is the
  only place raw JS is expected.
- **FastAPI backend**, Pydantic models for request/response shapes.
- No mobile app — web-first, rep-facing, internal tool (not
  public-facing), single sales org for Phase 1.

## Screens/components this design pass needs to cover

Straight from the MVP checklist (`mvp/README.md`), Step 3 scope:

1. **Login / authenticated-session gate** — email+password (not
   Google OAuth for login itself — that's a separate "Connect Google
   Account" action, see below). Nothing past this screen is reachable
   unauthenticated.
2. **Prioritized queue view** — the main screen. One row/card per
   lead, sorted by rank (see "Data shapes" below for the exact
   fields). This is what the system prompt's JSON output is
   structured for — see that shape below, it's the contract to design
   against.
3. **Missing-document checklist per lead** — which of
   application/bank-statements/prequal questionnaire are absent,
   sourced from `verify_drive_contents`.
4. **Recommended-action cards per lead** (call / text / email / info
   request) — inline draft review and edit, each individually
   approved. Approving a *call* specifically copies the phone number
   to clipboard and shows a confirmation — it never dials or opens any
   calling app. Approving a *text/email* actually sends it via
   Twilio/Gmail once approved.
5. **Back-office handoff review card** — Slack message draft, one of
   three types (`completion_handoff` / `info_request` /
   `urgent_callback_request`), same approval gate regardless of type.
6. **Inline spreadsheet editing with current-vs-proposed diff** —
   shown before any write to the source Google Sheet is confirmed.
7. **Communications search box** — search by name, company, email, or
   phone; returns matching email/SMS history. Real limitation to
   design around: a name/company search only returns email results,
   never SMS — Twilio's API can't free-text-search message bodies, so
   don't design a search box that implies SMS search works for
   non-phone-number queries.
8. **Call-outcome quick action** (answered / no answer / voicemail /
   didn't call) — **explicitly flagged as undesigned** in the
   decisions log (Decision 020's open item): "the interface design for
   prompting the rep at the right moment isn't done yet." This is
   probably the single most open question for this design pass — when
   and how does the UI prompt a rep to log a call outcome after
   they've placed a call outside the app entirely?
9. **"Connect Google Account" flow** — one-time OAuth consent
   (`drive.file` + `drive.readonly` + `gmail.send` + `gmail.readonly`
   scopes) plus the Google Picker widget so a rep selects which
   sheets/folders LeadPilot may access. Also the entry point for
   ad-hoc sheet lookups mid-session.

## Data shapes — what's actually available to bind the UI to

### The system prompt's queue output (what the agent produces each run)

```json
{
  "prioritized_queue": [
    {
      "lead_name": "String",
      "priority_tier": "Rank 1/2/3",
      "status_summary": "Context text",
      "missing_documents": ["List"],
      "recommended_actions": [
        {
          "type": "Call / Text / Email / Info Request / Spreadsheet Update",
          "tool": "initiate_lead_call / send_lead_text / send_lead_email / dispatch_slack_handoff / update_lead_sheet",
          "status": "AWAITING_REP_APPROVAL",
          "draft_content": "Tailored messaging block text",
          "diff": { "current": "String or null", "proposed": "String or null" }
        }
      ]
    }
  ],
  "pending_backoffice_handoffs": [
    {
      "lead_name": "String",
      "handoff_type": "completion_handoff / info_request / urgent_callback_request",
      "status": "AWAITING_REP_APPROVAL",
      "message": "String"
    }
  ]
}
```

### The underlying contact_history row (every staged/executed action)

| Field | Type | Notes |
|---|---|---|
| `event_id` | uuid | The identifier every approve/reject/execute action keys off |
| `lead_id` | uuid | Canonical lead (post-dedup) |
| `channel` | enum | `call` / `text` / `email` / `slack_handoff` / `sheet_edit` |
| `tool` | enum | Which tool produced this row |
| `stage` | enum | `drafted` / `awaiting_rep_approval` / `approved` / `executed` / `rejected` / `expired` |
| `timestamp` | datetime | Last stage-transition time |
| `rep_id` | uuid, nullable | Who approved/executed (null until approved) |
| `outcome` | enum, nullable | `delivered`/`failed` (texts/emails/Slack), or `pending`/`answered`/`no_answer`/`voicemail`/`didnt_call` (calls only) |
| `content_ref` | string | The drafted content (format varies by tool — see below) |
| `note` | string, nullable | Free text — also currently overloaded to carry `dispatch_slack_handoff`'s message type (known schema gap, see Open questions) |

### Per-tool output shapes (Marc's Group B — verified, from the actual code)

- **`get_contact_history(lead_id)`** → list of the contact_history rows
  above (as dicts), most recent first.
- **`initiate_lead_call(lead_id)`** → `{event_id, stage, phone_number}`.
  On approval, `execute_initiate_lead_call(event_id)` returns the
  phone number string (for the frontend to
  `navigator.clipboard.writeText()`) or `None` if not actually
  approved.
- **`dispatch_slack_handoff(lead_id, message_type, message)`** →
  `{event_id, stage, message_type, channel_ids}`. On approval,
  execute returns `{event_id, message_type, deliveries: [{channel_id, ok, ts}]}`
  — one delivery result per stakeholder channel (up to 3).
- **`send_lead_email(lead_id, subject, body)`** →
  `{event_id, stage, to, subject}`. On approval, execute returns
  `{event_id, to, message_id}`.
- **`send_lead_text(lead_id, message)`** →
  `{event_id, stage, to, message}`. On approval, execute returns
  `{event_id, to, message_sid, status}`.
- **`search_communications(rep_id, identifier)`** →
  `{identifier, emails: [...], texts: [...]}`. Each email:
  `{message_id, from, to, subject, date, snippet, has_attachment}`.
  Each text: `{message_sid, from, to, body, date_sent, direction}`.

### Group A tools (Abdoul's — `fetch_all_leads`, `update_lead_sheet`,
`verify_drive_contents`, `fetch_ad_hoc_sheet`, `log_call_outcome`)

**Not verified in this file** — built by the other developer, exact
field names not confirmed here. Use the system-prompt JSON shape
above as the contract for `fetch_all_leads`/queue data, and treat
anything more specific as needing a check against the actual code
before locking in a binding.

## Open questions worth putting to the designer explicitly

1. **Call-outcome logging UI** (see above) — genuinely undesigned,
   highest-value open question for this pass.
2. **`dispatch_slack_handoff` message type** currently lives in the
   `note` field, not a dedicated column — pending a schema decision
   with Abdoul. Design the three handoff types as visually distinct
   regardless of how the data ends up stored.
3. Google Picker integration point — not decided where in the
   dashboard "Connect Google Account" and per-sheet selection should
   live.

## Brand/visual guidance

None exists yet — no established color palette, typography, or
component library beyond "server-rendered Jinja2 + htmx." This is
open territory for the design pass, not a constraint to work around.
