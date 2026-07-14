# Interface build — decisions made autonomously during the Step 3 build

**Date:** 2026-07-14
**Built by:** Claude (Fable 5), on Marc's instruction to build the full
interface from `context-files/leadpilot_interface_design_spec_v001.md`
(+ `leadpilot_interface_design_context-v002.md`), answering any open
questions autonomously and logging every such answer here.
**Branch:** `marc-interface-build` in the `leadpilot` code repo.
**Status of each item:** provisional — Marc/Abdoul can reverse any of
these; each entry says what changing it would cost.

Marc pre-approved before the build started (not autonomous): full
10-screen scope now; queue backed by real routes hitting the existing
Step 2 tools (not mock JSON); reasonable defaults for the spec's open
items; theme/sound/panel-width prefs in localStorage (no
rep_preferences table yet).

---

## A. Product-behavior decisions

### A1. Approve = gate.approve() + tool execute, one click
The spec's "Approve-and-verb" buttons perform `gate.approve()` and the
tool's own `execute_*()` in a single request. The state machine
underneath is unchanged (drafted → awaiting_rep_approval → approved →
executed) — `approved` is passed through within one request rather
than being a separate rep-visible step. The tools' execute functions
still re-check `gate.try_execute()` themselves, so single-use holds
even if two approve requests race (covered by
`tests/test_ui.py::test_second_approve_is_a_noop_single_use`).

### A2. Failure policy: a post-flip execution failure keeps EXECUTED
If a tool raises *after* winning `try_execute()` (e.g. Twilio errors
mid-send), the UI commits the EXECUTED flip, sets `outcome=FAILED`,
and shows the failure on the card — it does NOT roll back to APPROVED.
Reason: a retry that could double-send is strictly worse than a lost
send for this product (0% duplicate contact is the named success
metric); recovery is a freshly staged draft, never a replay. Failures
*before* try_execute (config/validation) leave the row APPROVED and
the button available — nothing was consumed. This matches
update_lead_sheet's already-documented "approval was consumed"
semantics; the UI just applies it uniformly.

### A3. The UI persists provider delivery outcomes
`execute_send_lead_text/email/slack_handoff` return delivery results
but never wrote them anywhere. The approve route now sets
`outcome=DELIVERED` on success / `FAILED` on post-flip failure —
that's the Outcome enum's documented purpose for those channels, and
the approving caller is the natural integration point. No schema
change.

### A4. Fixed a real contract gap: executed calls now get outcome=PENDING
`log_call_outcome` refuses any row whose outcome isn't PENDING, and
the documented contract says an executed call sits at PENDING — but
nothing ever set it (`log_call_outcome`'s tests pre-set PENDING on
their fixture rows, masking the gap). `execute_initiate_lead_call`
now sets it when it wins the gate. Without this, §6f's outcome flow
would have been impossible end-to-end. One-line change in
`tools/initiate_lead_call.py`, commented at the site.

### A5. Google OAuth callback now redirects instead of returning JSON
`GET /auth/google/callback` is a top-level browser navigation arriving
back from Google's consent screen; it previously returned
`{"connected": true}` as raw JSON in the browser. It now 303-redirects
to `/?google=connected`. Error paths (401/400) are unchanged and the
existing tests only asserted those, so no test changes were needed.

### A6. Stale-write panel: spec's three exits collapsed to two buttons
§6e lists three exits (apply-my-edit-anyway / keep-new-value /
review-fresh-diff). "Apply my edit anyway" and "Review fresh diff"
are mechanically identical under the spec's own implementation
constraint — both stage a FRESH diff through the normal state machine,
and a fresh staged diff always requires review before approval, so
the "review" step is inherent, not a separate path. Rendered as two
buttons; the panel copy states nothing writes without one more
explicit approval. Restoring a third button is trivial if the
distinction matters to reps in practice.

### A7. "Who changed it + when" in the conflict panel: shown as unavailable
The spec wants the conflict panel to say who changed the cell. The
Sheets values API LeadPilot reads doesn't carry editor identity (that
needs the Drive Revisions API — a new scope/complexity question). The
panel says the value "changed outside this approval — editor identity
isn't available from the Sheets API" rather than guessing. Flag for a
future decision if reps need it.

### A8. Editable drafts: text, email, Slack only
Edit updates `content_ref` via the same atomic-conditional-UPDATE
discipline as gate.py (`WHERE stage='awaiting_rep_approval'`), so an
edit can never rewrite an already-approved/executed/rejected row.
Calls aren't editable (nothing to edit — the number comes from the
lead record); sheet diffs aren't editable (the diff is computed data —
reject and stage a fresh edit instead, which the inline edit form
makes cheap).

### A9. Interim rank heuristic until Step 4's agent exists
Rank 1 = answered call within 24h (the only "active interest" signal
LeadPilot itself stores today); Rank 2 = no executed contact; Rank 3 =
everything else, with the reason string distinguishing
unlogged-call / unanswered / stale-cadence cases. Every queue item
carries a human-readable `rank_reason` ("reps need to trust the
ranking, not just obey it"). Lives in `queue_builder.py` with the
same output shape Step 4's agent queue will use — swap the heuristic,
keep the templates.

### A10. Doc checklist matching (Decision 008 operationalized)
Required set: Application / Bank statements / Prequal questionnaire.
A file counts only if its name keyword-matches (application;
bank+statement; prequal|questionnaire), is strictly `.pdf`, and is
>5KB. A candidate that exists but fails a check is shown with the
reason ("not a PDF, doesn't count") rather than silently absent.
Keyword lists are two small tuples in `queue_builder.REQUIRED_DOCS`.

### A11. External deep links: Google Sheets only
§6g wants deep links ("Open in Twilio →"). contact_history doesn't
persist provider IDs (Twilio message SID, Gmail message id), and calls
never touch Twilio at all (Decision 016 — clipboard model), so only
the Google Sheets link is constructible (`source_id` IS the file id
under Decision 026). Persisting provider IDs is a schema question for
Abdoul — flagged, not done unilaterally.

### A12. Queue is org-wide; the unlogged-calls strip is rep-scoped
Leads are global by design (dedup across reps' sheets is Eval Case
2's whole point), so every rep sees the same queue in Phase 1's
single-org scope. The amber unlogged-calls strip filters to the
current rep's own executed calls — an unlogged call is the approving
rep's loose end, not a shared to-do.

### A13. Background pattern default: aurora (revises the quick-pick "off")
The pre-build quick answer said default-off. Building it surfaced the
§1 constraint that decides it: `backdrop-filter` glass needs something
behind it to blur — "over flat black, glass reads as gray," which
would make "None" silently defeat the decided design language. §3
already calls aurora the "default candidate — shows glass best."
Default is aurora at the spec's prototyped subtlety; "None" remains
one click away in the appearance popover. Sound default stays OFF; all
seven themes ship (trimming later = deleting CSS lines).

### A14. Hover/highlight states are accent-tinted (Marc, 2026-07-14)
Reverses the earlier white-alpha-only hover rule (part of the "accent
only on: rank pills, primary buttons, selected states, links, icons,
focus glow" constraint in Marc's glass system). Marc saw the accent
click-glow, liked it, and decided hover/highlight belongs on the
accent list too: generic buttons, queue items, tabs, and timeline
entries now hover with `rgba(var(--ar), ~0.08–0.12)` tint instead of
white-alpha. Exception kept deliberately: the Reject button's hover
stays semantic red — accent-tinting a destructive control would
misread, and semantic colors never re-theme. (Marc briefly asked for
Reject to hover accent too, then reversed within minutes and
confirmed the red exception stands.)

### A15. Designer's artifact ported 1:1 (2026-07-14)
`mockups/leadpilot_glass_reference.html` (the design LLM's export,
added by Marc) superseded all prose transcriptions of the glass
system — its values were ported verbatim into `app.css`. Confirmed
gaps the prose specs had lost: inset top-sheen on every surface,
Inter-first font stack, the full background stack (luminance ramp +
two viewport-relative accent ellipses + neutral white orb + corner
vignette), 1560px workspace ceiling with centered layout, segmented
tab control, accent-light card headers, neutral R2/R3 rank pills
(semantic colors reserved for meaning, not rank), amber
urgent-approve button, avatar in the lead header, hairline-separated
uppercase-label diff rows. Kept over the artifact (Marc's later
decisions, flagged for the designer): A14 accent-tinted hovers
(artifact hovers white-alpha) and drag-resizable panes (artifact
fixed at 240/280px — now our resize defaults). Also kept: the
background pattern picker (A13), with the artifact's accent-orb
layer as the default and the ramp/orb/vignette depth invariant
across all pattern choices.

## B. Implementation decisions

- **htmx 1.9.12 vendored** at `static/js/htmx.min.js` — no CDN
  dependency at runtime (internal tool, known browser floor unknown).
- **JS stays within the sanctioned exceptions** (§7): app.js contains
  clipboard copy, drag-resize, Web Audio clicks, and the glow-ring
  class toggle (tab active-state toggling folded under that same
  cosmetic-class exception). The Google Picker in the Connect drawer
  is the separately-sanctioned Google JS island. Clipboard auto-copy
  after approval is attempted on swap but never relied on — the
  confirmed card always shows the number and a manual "Copy again"
  button (browser transient-activation rules make auto-copy
  best-effort).
- **UI auth = same session chain, redirect semantics**:
  `require_rep_ui` runs the identical
  `auth.get_rep_for_signed_token` validation as the JSON API's
  `require_rep`, but full-page requests 303 to /login and htmx
  partials get 401 + `HX-Redirect` (so a pane never swaps in a login
  page).
- **External clients injected via module-level factories** in ui.py
  (tests monkeypatch them; None = tool builds its real client) — same
  DI pattern as the tools' own tests.
- **Seed script** `scripts/seed_demo_data.py` stages demo drafts
  through the REAL tool run() paths so approving them exercises the
  real gate; `--wipe` removes everything it created (including the
  per-rep run-lock row a Sync click leaves behind). Wipe before
  running the full pytest suite — some fetch_all_leads tests count
  all leads in the DB.
- **Slack demo channel IDs**: the seed sets in-process placeholder
  channel IDs if `SLACK_HANDOFF_CHANNEL_IDS` is empty, since
  dispatch_slack_handoff (correctly) refuses to stage with none
  configured.

## C. Verification

- `pytest`: 215 passed, 9 skipped (the pre-existing live-OAuth
  skips), including 22 new tests in `tests/test_ui.py` — auth gating,
  single-use approval through the UI path, no-send-after-reject,
  edit-before/after-execution, call→clipboard→outcome loop, urgent
  handoff fan-out, stale-write conflict + fresh-baseline re-stage,
  search truth-notices.
- Live: uvicorn + seeded data, exercised over HTTP — login, queue
  (rank pills, urgent sort, unlogged strip), lead center cards,
  approve-call → clipboard payload → outcome row → strip clears →
  lead re-ranks to R1, sync/docs/sheet tabs, both drawers.
- Not verifiable without a browser/human: drag-resize feel, Web Audio
  sounds, glass rendering, real Google Picker round trip. Marc should
  eyeball these first run.
