# LeadPilot — Interface Design Spec v001

**Status:** Agreed direction from Marc's design sessions with Claude
(claude.ai, 2026-07-14). Supersedes nothing — this is the first design
spec. Reads alongside `leadpilot_interface_design_context-v002.md`
(the *requirements* file); this file records the *decisions*.
**Owner sign-off:** Marc has approved everything marked "Decided."
Abdoul has not reviewed yet — items he should ratify are flagged.
**Suggested repo location:** `leadpilot-docs/design/` (new folder), or
alongside the interface context files if a design/ folder feels heavy.

---

## 1. Design language — Decided: glass

Dark-by-default, translucent "glass" surfaces. Metallic was built and
compared side by side and rejected as the base language: gradient
panel surfaces add visual weight to every card, and at
dozens-of-approvals-per-shift density that weight competes with the
draft text, which is the actual product. Metallic may return later as
one selectable theme (like Precog's "Metallic" candlestick style),
not the foundation.

This is LeadPilot's own language. It deliberately borrows from
Precog's sensibility (dark-first, instrument-adjacent, premium
restraint, CSS-token color system) but is not bound by Precog rules.
In particular it **intentionally breaks** Precog Dashboard's
"solid surfaces — no transparency, no backdrop-filter/blur" rule.
Record this as a conscious divergence (the way Precog documented the
TweakUI divergence), not an inconsistency.

### Core visual tokens

| Token | Value | Notes |
|---|---|---|
| Base background | `#0c0d10` | Near-black, warm-neutral |
| Panel surface | `rgba(255,255,255,0.035)` + `backdrop-filter: blur(16–18px)` | The glass |
| Card surface (nested) | `rgba(255,255,255,0.03)` | Slightly quieter than panels |
| Hairline border | `1px solid rgba(255,255,255,0.08)` | 0.07–0.12 range by emphasis |
| Text primary | `#e9e7e2` (near-white warm) / `#fff` for names+headings | |
| Text dim | `#9a9ea6` | Supporting copy |
| Text faint | `#5b6067` | Labels, timestamps |
| Radius | 12px panels/cards, 8px buttons/rows/inputs | |
| Micro-labels | 10–12px, uppercase, letter-spacing 0.08–0.15em | Instrument nod without full monospace |
| Success | green family (`#5fd6a2` on dark) | Sent/confirmed states |
| Warning/urgent | warm amber (`#f0a56b`) | Urgent handoffs, stale writes, pending outcomes |
| Danger | soft red (`#ff7b7b`) | Old value in diffs, missing docs |

Accent color is never hardcoded per component — everything
accent-touched (pills, primary buttons, selection, click glow, links,
resize-handle hover) derives from one CSS custom property, so the
theme system is free.

### Requires a base layer decision (open)

`backdrop-filter` only reads as glass when there's something behind
it to blur. The app needs a subtle base layer beneath the panels —
see Background patterns below. Over flat black, glass reads as gray.

## 2. Color themes — Decided: user-selectable, CSS-variable driven

Seven themes prototyped; ship list can be trimmed:

| Theme | Accent RGB | Light-accent text |
|---|---|---|
| Cool blue (default) | 59,130,246 | `#9cc3f9` |
| Deep blue | 30,79,163 | `#7fa4dd` |
| Royal purple | 108,79,209 | `#b3a0ef` |
| Dark purple | 67,48,143 | `#9c8cd6` |
| Teal | 29,158,117 | `#7fd6b4` |
| Gold | 201,162,39 | `#e3c96a` |
| Slate (near-mono) | 125,131,140 | `#c7cad0` |

One variable (`--ar`, the accent as an RGB triple, so it composes into
`rgba(var(--ar), α)`) plus one derived light-accent for text on dark.
Semantic colors (success green, warning amber, danger red) do NOT
re-theme — they mean the same thing in every theme.

## 3. Background patterns — Decided: subtle, theme-tinted, optional

Four prototyped: aurora orbs (default candidate — shows glass best),
fine grid, diagonal weave, corner glow. All tinted from the active
theme accent at low opacity (≈0.08–0.25 in radial falloff).

**Legibility rule:** this screen is read for hours. Patterns stay at
prototyped subtlety or lower. Open question for Marc/Abdoul: whether
default is a subtle pattern or "None" with patterns as opt-in
personalization.

## 4. Layout — Decided: three-pane resizable workspace

Not separate windows — a multi-panel workspace (Superhuman/Linear/
Attio pattern), honest to what htmx partial swaps can do:

- **Left (≈200px):** prioritized queue. Rank pills, one-line status.
- **Center (flex):** selected lead — header, then stacked action
  cards (text / email / call / sheet edit), each with its own
  approve/edit/reject.
- **Right (≈250px):** context rail — documents checklist + contact
  history timeline, always visible, swappable content.
- **Drag-to-resize** between panes (5–6px invisible handles,
  accent-tint on hover, min-width floor ≈130px). Vanilla JS —
  second sanctioned raw-JS exception alongside clipboard copy.
  **Persist widths per rep** (localStorage or rep prefs — decide).
- **Quick-switch tabs** above the workspace: Queue / Sheet / Docs.
  Swap only the center pane; queue and history never disappear.
- **Slide-over drawers** (not new pages, not modals) for occasional
  flows: communications search, ad-hoc sheet lookup, Connect Google
  Account.
- **True modals reserved** for rare high-stakes moments only
  (stale-write conflict). Never for routine approvals — modals at
  volume train click-through.

## 5. Interaction feedback — Decided

Every clickable element confirms registration instantly:

- **Press:** scale(0.96) on :active, ~80ms.
- **Registered:** brief accent glow ring
  (`box-shadow: 0 0 0 1px rgba(--ar,0.7), 0 0 12px rgba(--ar,0.3)`,
  ~180ms). Chosen over background flash — reads far better on dark
  glass.
- **Approve buttons transition to a persistent confirmed state**
  ("Sent ✓" in success green, disabled) — the rep never wonders
  whether a click landed.

### Sound themes — Decided

Per-rep selectable click sounds, synthesized via Web Audio (no audio
files). Prototyped set: glass tap (default candidate), soft chime,
mechanical click, marimba, low pulse, crystal ping, off.

**Open decision before ship:** default should probably be OFF or
"match system" — a bullpen of reps with sound on is a real
environment problem. Marc flagged, not yet ruled.

## 6. Per-screen decisions

### 6a. Login gate
Plain email + password card, centered, minimal. One clarifying line:
"Google connects later, inside the app." Deliberately NO Google
button on this screen — login is DB-backed (Decision 013) and putting
OAuth here invites reps to conflate login with the separate Connect
Google Account consent (Decision 026).

### 6b. Connect Google Account + Picker
Two-step in one panel: (1) consent explainer with three plain-language
scope chips — "Sheets you pick / Folders you pick / Send email as
you" — and the line "your edits show up in Google's revision history
under your own name" (Decision 026's attributability, framed as a rep
benefit); (2) after connect, the Picker list with multi-select,
grant button disabled until ≥1 item picked, and a confirmation naming
exactly what was granted and that hourly + ad-hoc runs now cover it.
Lives as a slide-over; also the entry point for `fetch_ad_hoc_sheet`.

### 6c. Action cards (text / email / call)
Shared anatomy: uppercase channel micro-header with icon, draft body,
button row (Approve-and-verb primary, Edit, Reject). Approve is the
only accent-filled button on a card. Email cards show the subject
line above the body. Call cards state before approval: "Approving
copies the number to your clipboard. Nothing dials automatically."

### 6d. Slack handoff cards
Three types visually distinct via badge + (for urgent) warm amber
border and header — but IDENTICAL approval anatomy. Urgency changes
prominence and sort order, never the gate (non-negotiable, restated
here so no future design pass re-litigates it). Every card carries
"Posts to 3 back-office stakeholders" so the rep sees who receives it
before approving — the what-to-whom confirmation principle without a
scary modal.

### 6e. Sheet diff + stale-write recovery (Decision 034 UI)
Diff: label column + struck-through old value (red) + proposed value
(green), approve/reject below.
Stale-write conflict (the rare sanctioned modal/prominent panel):
1. Reassurance FIRST: "Your approved write was held — nothing was
   overwritten."
2. Three-row comparison: what you approved / what the sheet now says
   (with who changed it + when — matters in multi-rep overlap) /
   the original value.
3. Three exits: **Apply my edit anyway** (stages a FRESH diff against
   the new baseline for one more confirmation — never a silent
   re-apply), **Keep the new value** (discard held write), **Review
   fresh diff**.

**Implementation constraint:** "Apply anyway" routes back through the
normal drafted → awaiting_rep_approval → approved state machine as a
NEW staged write with the new baseline. Not a retry of the old row.
Zero special-case code; Decision 034's guarantee stays intact.

### 6f. Call-outcome logging — Decided (resolves Decision 020's open item)
Two-moment design, both approved by Marc:

- **Moment 1 — capture at the call.** The instant approve-and-copy
  fires, the same card becomes a four-button outcome row: Answered /
  No answer / Voicemail / Didn't call. One tap, inline, NON-BLOCKING —
  the rep can dial and move on without logging.
- **Moment 2 — persistent queue nudge.** Unlogged calls accumulate in
  an amber strip in the queue with the same four inline buttons per
  call. The strip states the consequence, not the ask: "Rank 3
  follow-ups pause for these leads until logged" — stalled pipeline
  motivates reps; data hygiene doesn't.
- **History timeline renders pending as amber "outcome unknown —
  follow-up logic paused,"** visually distinct from a logged
  "no answer." A hole in the record is not neutral.
- **Ratified: NO end-of-day forced modal.** Considered and rejected —
  punishes the rep at departure, trains resentment. The persistent
  strip applies the same completeness pressure without a wall.
- **Ratified: "Didn't call" is a first-class outcome**, equal in the
  row. Copy-then-don't-dial is common; one tap keeps the record
  honest instead of leaving phantom pendings.

### 6g. Contact history timeline
One chronological stream across all channels (call/text/email/Slack/
sheet edit) — never per-channel tabs. Every entry is CLICKABLE:
expands inline with full content where LeadPilot stores it
(texts, emails, sheet-edit diffs); where it doesn't (calls), shows
what's known plus an external deep link ("Open in Twilio →" /
"Open in Google Sheets →"). Shows actor ("· you" / "· Marc") per
Decision 034's multi-rep overlap scenario. Reachable from both the
queue (in context) and search (out of context).

### 6h. Communications search
Slide-over. Input accepts name / company / email / phone. Result:
identity header (avatar, name, company, identifiers, "Open lead"
button) + unified result rows. **The SMS limitation is designed into
the UI:** the hint line under the box states "Phone numbers search
texts and email · names and companies search email only," and a
name-search result shows an amber "email results only — texts can't
be searched by name" notice. Truth in the interface beats silent
partial results.

## 7. Feasibility notes for the Step 3 build (htmx reality check)

- Panel swaps, tab switches, card state transitions, inline expands:
  all natural htmx partial swaps. Nothing above needs client-side
  state management.
- Sanctioned vanilla-JS exceptions now number FOUR (was one):
  1. `navigator.clipboard.writeText()` (already sanctioned)
  2. Panel drag-resize
  3. Web Audio click sounds
  4. Click-feedback micro-animations (CSS-only where possible; JS
     only for the glow-ring class toggle)
  If Abdoul objects to any, resize and sound are the cuttable ones —
  feedback and clipboard are not.
- The Google Picker is Google's own JS widget — it was always going
  to be a JS island; contain it in the Connect slide-over.
- `backdrop-filter` has good modern-browser support; internal tool +
  known browser population = fine, but confirm the sales org's
  browser floor once known.

## 8. Open items out of this design pass

1. **Base-layer/pattern default** — subtle pattern vs. None by
   default (§3). Marc + Abdoul.
2. **Sound default** — off vs. match-system vs. glass-tap (§5).
   Leaning off/quiet; not ruled.
3. **Panel-width persistence** — localStorage vs. rep prefs table.
4. **Ship list of themes** — all seven, or trim (§2).
5. **Abdoul ratification** of: the four JS exceptions (§7), the
   stale-write "new staged write" implementation constraint (§6e),
   and this spec generally.
6. **Google Picker placement** was an open decision in
   `decisions/README.md` — this spec answers it (Connect slide-over,
   §6b); the decisions log should be updated to point here when this
   file lands in the repo.

## 9. Provenance

Prototyped as interactive mockups in Marc's claude.ai design sessions,
2026-07-14: three-pane workspace (v1–v3), theme lab with glass vs.
metallic comparison, handoff + stale-write flows, call-outcome
logging, login/connect/search. Decisions recorded here were approved
by Marc in-session; nothing in this file is speculative except items
explicitly marked open.
