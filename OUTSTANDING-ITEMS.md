# Outstanding items — prioritized for the class project (demo/functionality focus)

**Reframed 2026-07-16.** This project is currently a class project, not
headed toward a real launch — compliance, legal, governance, and
production-ops concerns don't matter for grading or a presentation
demo right now. Priority is: does it work end-to-end, does it look and
feel good to an end user, is it reliable enough to survive a live
demo, and does the presentation land well. Everything from the earlier
production-launch framing is kept at the bottom (Tier 5) rather than
deleted — pick this back up if the project ever becomes a real
product, but it's not the list to be working from today.

Every item links back to where it's actually documented in more detail
(`testing/known-issues-log.md`, `decisions/README.md`,
`design/interface-build-decisions-v001.md`). Update those source docs
when an item resolves; update this file's status alongside them.

---

## Tier 1 — Demo-day functionality risks (fix or verify before presenting)

1. ~~**Bug: `scripts/seed_demo_data.py --wipe` crashes.**~~ — **fixed
   2026-07-16.** Reproduced directly: `IntegrityError
   (ForeignKeyViolation) — update or delete on table "reps" violates
   foreign key constraint "rep_google_credentials_rep_id_fkey"`. The
   wipe path deleted `RepSession`/`AgentRunLock` rows before deleting a
   rep, but never deleted that rep's `rep_google_credentials` row
   (left behind by a real "Connect Google Account" flow, e.g. for a
   live-Sheets walkthrough) — same FK-ordering issue the two existing
   deletes were already written to avoid, just missing one table.
   Fixed in `scripts/seed_demo_data.py`'s `wipe()`: added a
   `RepGoogleCredential` delete before `session.delete(rep)`, same
   pattern as the existing `AgentRunLock`/`RepSession` deletes.
   Verified: `--wipe` now runs clean.
2. ~~**Full pytest suite had 1 failure.**~~ — **confirmed fixed
   2026-07-16.** The `test_detect_changes_against_real_data` failure
   was exactly the stale-seed-data issue item 1 caused (a duplicate
   `(source_id, row_ref)` key in `lead_source_rows` left over from a
   demo run `--wipe` couldn't clean up). Reran the full suite after the
   fix: **293 passed, 11 skipped, 0 failed.** (Skip count rose from 2 to
   11 versus the pre-wipe run — expected, since wiping the demo rep
   also removed its `rep_google_credentials` row, so the live-OAuth
   tests that depend on a connected rep now correctly skip rather than
   run against stale/absent credentials.)
3. **Twilio trial-account SMS block (Issue 005).** `send_lead_text`
   will fail live against a real number unless the Twilio account is
   upgraded off its trial tier first — confirmed root cause: trial
   accounts reject custom SMS bodies outright (`572006`). Decide now,
   before presenting: upgrade the account (cheap, immediate), or plan
   the demo so it doesn't attempt a live text send (e.g. narrate the
   drafted text and approval flow without clicking through to the
   actual Twilio call). Don't find out live on stage.
4. **Group B tools have never been run through a live, OAuth-connected
   round trip.** `send_lead_email`, `dispatch_slack_handoff`,
   `search_communications`, `initiate_lead_call`, and `send_lead_text`
   are unit-tested against fakes only — only Group A's tools
   (`fetch_all_leads`, `update_lead_sheet`, `verify_drive_contents`,
   `fetch_ad_hoc_sheet`) have a proven live end-to-end pass. Do one
   full live click-through of every approve button — call, text,
   email, Slack handoff, search — before presenting, so nothing
   surprises you in front of an audience.
5. **Known UX bugs from Marc's live walkthrough that directly affect
   what a demo can show** (all still open, all "future version" —
   grouped since they share a root cause: Google Drive/Picker
   per-item-grant limitations):
   - `.xlsx` files look pickable in the Picker but fail to read (Issue
     008) — make sure whatever sheet you demo from is a real native
     Google Sheet, not an Excel export.
   - Files sitting loose in Drive root are invisible to the
     folder-grant flow (Issue 009) — make sure any demo "deal
     documents" live inside an actual folder you grant, not the Drive
     root.
   - Status baked into a sheet's row background color isn't inferred
     yet, only legend-color *display* is (Issue 011) — if your demo
     sheet encodes status purely by color with no Status column text,
     leads will show blank status. Either populate a real Status
     column on the demo sheet, or plan to narrate this as a known,
     scoped limitation rather than have it look broken.
   - Google Picker renders bright white regardless of the app's dark
     theme (Issue 010) — confirmed structural, not fixable quickly.
     Just don't be caught off guard by the visual jump mid-demo.
6. **Do one full dry run of the demo/story-mode script
   (`scripts/stage_demo_scenario.py`) end-to-end, on the actual machine
   you'll present from, close to presentation time.** This is the
   single highest-value thing on this whole list — it's what will
   actually surface anything items 1-5 above didn't catch.

## Tier 2 — UX polish (worth doing, meaningfully improves how it reads to an audience)

- **Rank pills (R1/R2/R3) have no tooltip explaining what they mean.**
  An audience seeing a bare "R2" with no explanation reads as
  confusing (this came up earlier in review). Cheap, high-value fix:
  add a `title` attribute to the rank pill the same way the pipeline-
  status badge next to it already has one — reuse `rank_reason`, which
  every queue item already carries, as the tooltip text.
- Stale-write conflict panel says "editor identity isn't available"
  rather than naming who changed a cell (build-decisions A7) — minor;
  only worth polishing if a stale-write demo path is actually planned.
- Provider deep links ("Open in Twilio →" etc.) only work for Google
  Sheets today, not Twilio/Gmail (A11) — cosmetic gap, not worth
  building just for a demo.
- `expire_stale_drafts()`'s 7-day staleness threshold — irrelevant at
  demo timescale, no action needed before presenting.

## Tier 3 — Reliability foundations already solid (nothing to fix — good material to cite in the presentation)

These are genuinely presentable engineering results, not just "not
broken" — worth mentioning directly when demoing or writing up the
project:

- Approval-gate single-use enforcement proven under real concurrency —
  10 simultaneous execute attempts against the same approved row,
  exactly one wins, every time (`test_gate.py`).
- Same-cell concurrent spreadsheet writes can't silently clobber each
  other (Decision 034) — has a real stale-write recovery UI panel, not
  just a backend check.
- A real security review (Decision 038) found and fixed two actual
  prompt-injection bypasses (zero-width Unicode, homoglyph
  substitution) by testing the exploit directly, not just reading the
  code — good evidence of engineering rigor for a class presentation.
- Duplicate-contact prevention (the PRD's named 0% duplicate-contact
  goal) is enforced structurally via atomic locks, not a "best effort"
  check.

## Tier 4 — Functionality not built, and not needed for this demo

Skip these entirely for now — they'd matter for a real product, not
for demoing what's built to a class:

- `GoogleDocsConnector`, Excel/OnlyOffice/LibreOffice connectors —
  unscheduled, no demo depends on them.
- Anything from the TCPA/CAN-SPAM/DNC compliance research — real
  regulatory concerns for a real product, not relevant to a class
  project demo.

---

## Tier 5 — Deferred: production-launch concerns (not priority right now, kept for later)

Everything below assumed this was heading toward real deployment. It
isn't right now — revisit only if this project continues past the
class context into something that will actually contact real leads.

**Compliance/PII** (would block real-lead contact): consumer-vs-
business lead classification, suppression/opt-out table, inbound
Twilio STOP-reply webhook, CAN-SPAM email footer, consent provenance
fields, National DNC Registry scrub, `search_communications`
retention review, unmasked SSN/EIN/DOB in `lead_source_rows.raw_data`.
Full detail: `compliance/tcpa-can-spam-dnc-research.md`,
`testing/known-issues-log.md` Issues 004/007/012.

**Governance**: real written partnership agreement (50/50 equity and
fork rights are only confirmed verbally). `governance/README.md`.

**Production ops**: connecting the Render repo + setting secrets,
confirming the secrets-management story is final, Twilio account
cleanup (delete `supporttest.py`, rotate credentials), building actual
monitoring/alerting (currently a draft checklist), writing a real
release/rollback runbook, soft launch to one real rep.
`mvp/README.md` Step 5, `monitoring-observability/README.md`,
`release-process/README.md`, `testing/known-issues-log.md` Issue 005.

**Small schema/design questions with no urgency either way**: whether
Gmail shares the drive.file OAuth client, persisting provider message
IDs, Drive Revisions API for "who changed it," narrower alternative to
the `drive.readonly` scope. `decisions/README.md`'s "Open decisions"
section, build-decisions log.

**Doc hygiene**: already done 2026-07-16 — see git history on
`tech-stack/stack-overview.md` and `security/threat-model.md`.

## Notes

- This file is a snapshot as of 2026-07-16. Re-derive rather than
  trust it once stale — check the linked source docs directly.
- If priorities shift back toward a real launch, Tier 5 is the
  starting point — it's the full compliance/governance/ops backlog
  from before this reframe, not abridged.
