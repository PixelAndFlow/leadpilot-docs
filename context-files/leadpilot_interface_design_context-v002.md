# LeadPilot — Interface Design Context v002

Paste this at the start of a session with a design-focused LLM to get
help designing/wireframing LeadPilot's Step 3 rep-facing dashboard.
Supersedes `leadpilot_interface_design_context-v001.md` — v001 kept
unedited for reference. This version adds a rep's-day workflow
narrative, a dedicated section on why contact history is the product's
emotional core (not just a data feature), a "what matters to a
salesperson" section, and verified data shapes for the tools v001
flagged as unconfirmed. Distinct from the general
`leadpilot_context-v00X.md` project-restore file — this one is scoped
tightly to what an interface designer needs.

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

Backend is functionally done but **split across two unmerged
branches**: all 11 agent tools exist and are tested, six built by
Marc, five by Abdoul, plus a same-cell write-conflict fix (Decision
034) layered on top. **None of the interface described below exists
yet** — this file is context for designing it from scratch. Tech stack
is locked (see below), not up for debate in this design pass.

## A rep's day — the workflow this interface has to support

Concrete, not abstract, because the screens should be designed around
this sequence, not around the tool list:

1. **Rep logs in** and opens the dashboard. They did not open a
   spreadsheet first — the whole point is they never need to again.
2. **The queue is already built** (the hourly agent run already
   happened): a ranked list of leads pulled from every sheet this rep
   personally connected, each with a status summary and a recommended
   next action already drafted.
3. **Rep scans the queue**, top to bottom by rank. For each lead, they
   need to answer, in seconds, without clicking in: *who is this, why
   are they ranked here, what happened last time I touched them, and
   what does the agent want me to do now?*
4. **Rep opens a lead** to review the draft in full — a text message,
   an email, a call recommendation, or a document-gap summary. They
   may edit the draft. They approve or reject.
5. **On approval**, the real thing happens: a text sends, an email
   sends, a phone number lands on their clipboard so they can dial in
   their own phone/softphone, or a Slack message posts to back-office.
6. **If they placed a call**, at some point — mid-flow or afterward —
   they need to report what happened (answered / no answer / voicemail
   / didn't call) so tomorrow's queue can act on it. **This step has no
   designed UI yet — see Open Questions.**
7. **Rep occasionally gets an inbound call or text from an unknown or
   half-remembered number** and needs to search — by phone, email,
   name, or company — to instantly pull up who this is and every prior
   email/text exchange before picking up or replying.
8. **Rep occasionally corrects a field directly** (a status update, a
   typo) — sees a before/after diff, confirms, and the real source
   Google Sheet updates, attributed to them.
9. **Rep occasionally hands LeadPilot a brand-new sheet** they just
   received, for an immediate one-off look, without waiting for the
   next hourly cycle.

The design should optimize hardest for steps 3-5 — that loop repeats
dozens of times a day and is where the product either saves real time
or becomes one more screen reps route around.

## Why contact history is the product's emotional core, not just a feature

This deserves more design attention than a "history" tab would
normally get. The Problem statement this product exists to solve is
explicitly about contact history: reps currently burn **over 2.5
hours a day** manually cross-referencing timestamps across scattered
sheets just to avoid two very real failure modes, and the interface
has to make both failures structurally hard to have, not just possible
to check for:

- **Over-dialing an already-contacted lead.** Nothing damages trust
  with a prospect faster than a rep who clearly doesn't remember
  calling them yesterday. This is why `LeadActionLock` exists at the
  data layer — but the *rep* also needs to see this at a glance before
  they approve an action, not just be silently blocked by the backend.
- **Losing a warm lead who needed a multi-channel nudge.** If a call
  goes unanswered, the rank-3 prioritization logic depends on the
  agent knowing that and staging a follow-up text or email. If the rep
  never sees *why* a lead is ranked where it is, they won't trust the
  ranking, and they'll go back to manually re-checking sheets in
  parallel — which defeats the product.

Beyond avoiding mistakes, contact history is what lets a rep sound
competent on a call: picking up and instantly knowing "we texted about
bank statements Tuesday, they haven't replied" is the difference
between a rep who sounds prepared and one who sounds like a stranger.
Concretely, the history view needs to:

- Show **every channel in one chronological timeline** — call, text,
  email, Slack handoff, and sheet edits — not separate tabs per
  channel. A lead reached by phone once and email twice needs to read
  as one continuous relationship, not three disconnected logs.
- **Make the gap visible when a call's outcome hasn't been reported
  yet.** A call that was placed but never logged (`outcome: pending`)
  is a hole in the record, not neutral — the UI should distinguish
  "we don't know what happened" from "no answer," since the former
  silently breaks the rank-3 follow-up logic for that lead.
- **Surface who took the action**, not just what happened — multiple
  reps can end up connected to overlapping leads (see Decision 034's
  same-cell-write scenario), so "which of us actually said what" can
  matter.
- Be reachable from **both** the queue (per-lead, in context) and the
  standalone communications search box (by identifier, out of
  context) — a rep sometimes starts from "this lead in front of me"
  and sometimes from "this phone number just called me."

## What else matters to a salesperson, specifically

Things worth designing for even though they're not in a tool's
input/output schema:

- **Every screen costs selling time.** The success metric this whole
  product is judged on is minutes of admin time per day, not feature
  count. If approving a draft takes more than a couple of clicks, or
  requires reading through unnecessary chrome, that's a design defect
  even if it's "just one more step."
- **Reps need to trust the ranking, not just obey it.** >90% adherence
  to the recommended order is a named success metric — that only
  happens if a rep can see *why* a lead is Rank 1 (active interest in
  the last 24 hours) versus Rank 3 (stale, needs a cadence nudge)
  without digging. Treat the ranking as something to explain, not a
  black-box sort order.
- **Drafts need to feel like something the rep would actually send.**
  A generic-sounding auto-text undermines confidence in the whole
  system fast. The interface should make editing a draft before
  approval effortless — not buried behind an extra click — since a rep
  who feels the copy is "off" needs a fast path to fix it, not a
  reason to reject and do it manually outside the tool.
- **Approval has to feel meaningfully different from a "next" button.**
  Because financial-adjacent, relationship-sensitive actions (a text
  to a real prospect, a spreadsheet write) are one click away from
  firing for real, the UI needs the confirmation moment to clearly
  state *what* is about to happen to *whom* — without turning every
  approval into a scary modal that trains reps to click through
  without reading. This tension (careful vs. fast) is a real design
  problem worth naming to the designer, not a solved one.
- **Reps process a high volume of leads per day.** Interactions that
  work fine once (a multi-step modal, a page reload) get painful at
  volume. Design with the assumption this screen gets used dozens of
  times per shift, not occasionally.
- **Income is tied to conversions, not to using the tool correctly.**
  If the dashboard is slower than working the spreadsheet directly,
  reps will route around it — this isn't a hypothetical, it's the
  exact failure mode the product's own success metrics are trying to
  prevent. Speed and trust are not nice-to-haves here.

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
  the `initiate_lead_call` approval flow — this is the only place raw
  JS is expected.
- **FastAPI backend**, Pydantic models for request/response shapes.
- No mobile app — web-first, rep-facing, internal tool (not
  public-facing), single sales org for Phase 1.

## Screens/components this design pass needs to cover

1. **Login / authenticated-session gate** — email+password (not
   Google OAuth for login itself — that's a separate "Connect Google
   Account" action, see below). Nothing past this screen is reachable
   unauthenticated.
2. **Prioritized queue view** — the main screen, and where reps spend
   most of their time (see "A rep's day" above). One row/card per
   lead, sorted by rank — see "Data shapes" below for the exact fields.
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
   shown before any write to the source Google Sheet is confirmed. As
   of Decision 034, a write can also be rejected as stale (the cell
   changed since the diff was shown) — the UI needs a path for "show
   me the fresh value and let me re-decide," not just a generic error.
7. **Contact history timeline** (see the dedicated section above) —
   both embedded per-lead in the queue and reachable via search.
8. **Communications search box** — search by name, company, email, or
   phone; returns matching email/SMS history. Real limitation to
   design around: a name/company search only returns email results,
   never SMS — Twilio's API can't free-text-search message bodies, so
   don't design a search box that implies SMS search works for
   non-phone-number queries.
9. **Call-outcome quick action** (answered / no answer / voicemail /
   didn't call) — **explicitly flagged as undesigned** in the
   decisions log (Decision 020's open item). This is probably the
   single most open question for this design pass — when and how does
   the UI prompt a rep to log a call outcome after they've placed a
   call outside the app entirely?
10. **"Connect Google Account" flow** — one-time OAuth consent
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
| `content_ref` | string | The drafted content (format varies by tool) |
| `note` | string, nullable | Free text — also currently overloaded to carry `dispatch_slack_handoff`'s message type (known schema gap, see Open questions) |

### Per-tool output shapes — verified against the actual code on both branches

- **`get_contact_history(lead_id)`** → list of the contact_history rows
  above (as dicts), most recent first.
- **`initiate_lead_call(lead_id)`** → `{event_id, stage, phone_number}`.
  On approval, execute returns the phone number string (for
  `navigator.clipboard.writeText()`) or `None` if not actually
  approved.
- **`dispatch_slack_handoff(lead_id, message_type, message)`** →
  `{event_id, stage, message_type, channel_ids}`. On approval, execute
  returns `{event_id, message_type, deliveries: [{channel_id, ok, ts}]}`
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
- **`fetch_all_leads(rep_id)` / `fetch_ad_hoc_sheet`** → list of rows,
  each `{lead_id, source_id, row_ref, name, phone, email, company, status}`
  — this is the per-lead shape the queue view's rows ultimately trace
  back to.
- **`verify_drive_contents(rep_id, folder_id)`** → list of
  `{file_id, name, mime_type, size_bytes, created_time}` — the raw
  material the "missing documents" checklist is computed from (compare
  against the expected application/bank-statements/prequal set).
- **`update_lead_sheet` staging** → `{event_id, status, field, current, proposed}`
  — this is the diff shown before approval. On approval, execute
  returns `{executed: true, source_id, row_ref, field, value}`, or
  `{executed: false}` if the approval was never actually won. As of
  Decision 034, execute can also raise an error if the cell changed
  since the diff was shown — the UI needs to handle that as "refresh
  and show me the new diff," not a generic failure.
- **`log_call_outcome(event_id, outcome, note)`** → `{event_id, outcome, note}`.

## Open questions worth putting to the designer explicitly

1. **Call-outcome logging UI** (see "A rep's day," step 6) — genuinely
   undesigned, highest-value open question for this pass. Consider:
   does this live as a persistent nudge in the queue for any call
   still `outcome: pending`, a prompt immediately after the clipboard
   copy, or something else entirely?
2. **`dispatch_slack_handoff` message type** currently lives in the
   `note` field, not a dedicated column — pending a schema decision.
   Design the three handoff types as visually distinct regardless of
   how the data ends up stored.
3. **Google Picker integration point** — not decided where in the
   dashboard "Connect Google Account" and per-sheet selection should
   live.
4. **Stale-write recovery flow** (new, Decision 034) — when an
   approved spreadsheet edit turns out to conflict with a change made
   since the diff was shown, what does the rep see, and how do they
   get to a fresh diff without losing their edit?
5. **Approve-fast vs. read-carefully tension** (see "What else matters
   to a salesperson") — no answer yet, worth exploring a few
   directions rather than picking one by default.

## Brand/visual guidance

None exists yet — no established color palette, typography, or
component library beyond "server-rendered Jinja2 + htmx." This is
open territory for the design pass, not a constraint to work around.
