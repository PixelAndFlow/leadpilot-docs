# LeadPilot — Context File v002

Paste this at the start of a new AI session to restore full context.
Supersedes v001 — v001 kept unedited for reference.

## What LeadPilot is

An AI agent for B2B Sales and Business Development teams that
orchestrates lead triage and multi-channel communication pipelines
across Google Workspace (Sheets, Drive) and Slack. Runs autonomously
on a persistent hourly schedule. Owners: Marc Delsoin, Abdoul Ba.

## What changed in v002 (PRD v1 → v1.01)

- **Nothing fires automatically anymore.** The agent only drafts —
  outreach (call/text/email), the back-office handoff, and spreadsheet
  edits all sit as `AWAITING_REP_APPROVAL` until the rep explicitly
  confirms. This resolves the "auto-send vs. rep-approval" question
  that was open in v1.
- **Unified interface** now also supports writing rep-approved edits
  back to the source Google Sheets (not just viewing the queue), so
  reps don't leave the interface to update a lead.
- **Back-office handoff** can now be either a phone call
  (`initiate_backoffice_call`, new tool) or a predetermined Slack
  message (`dispatch_slack_handoff`, extended) — rep picks, or reviews
  whichever the agent proposed.
- **New tool `search_communications`** — searches a client's email/text
  history by any known identifier (email, phone, full name, company
  name), returning history, attachments, and confirmed documents.
- **New tool `update_lead_sheet`** — writes back to the source sheet,
  but only after the rep confirms a shown current-vs-proposed diff.
- **Access control** — only authenticated, pre-authorized rep accounts
  can trigger the agent, view lead data, or approve staged actions.

Full detail: `prd/LeadPilot_PRD_v1.01.md`.

## The problem it solves

Outbound sales reps lose ~2.5 hours/day cross-referencing scattered
lead spreadsheets and contact histories to avoid duplicate outreach,
and deal handoffs to back-office teams are delayed ~4 hours because
notifying the 3 relevant stakeholders on Slack is manual.

## The seven tools

1. `fetch_all_leads` — Google Sheets API, pulls raw lead rows across
   designated spreadsheets
2. `get_contact_history` — Google Voice API / call log tracker,
   returns timestamped contact attempts across call/text/email
   (NOTE: this API's actual existence/shape needs verification — see
   Open Issue 001 below)
3. `verify_drive_contents` — Google Drive API, checks file presence,
   type, size for application/bank-statement/prequal documents
4. `dispatch_slack_handoff` — Slack Web API `chat.postMessage`, drafts
   a handoff or info-request message to the 3 back-office stakeholder
   accounts. **Stages only** — fires only after rep approval
5. `search_communications` *(new)* — searches email/text history by
   any known contact identifier (email, phone, name, company);
   read-only, no approval gate needed
6. `update_lead_sheet` *(new)* — writes a rep-approved edit back to the
   source Google Sheet after the rep confirms a current-vs-proposed
   diff
7. `initiate_backoffice_call` *(new)* — proposes a call to a
   back-office stakeholder as an alternative to the Slack handoff;
   stages only, places the call only after rep approval

## Prioritization logic (system prompt v1.01)

- Rank 1: active interest within last 24 hours
- Rank 2: new uncontacted leads
- Rank 3: old leads needing multi-channel cadence follow-up
- After ranking: verify Drive contents (application, 3 months bank
  statements, prequal questionnaire); draft the recommended next
  action per lead (call/text/email/info-request) as
  `AWAITING_REP_APPROVAL`; if docs are complete, draft a back-office
  handoff (call or Slack) also `AWAITING_REP_APPROVAL`. The agent
  never sends, calls, or writes on its own — a rep-triggered
  confirmation in the interface is required for every side-effect
  action.

## Security posture (this is the most-developed area of the project)

Primary threat: indirect prompt injection via malicious text in a
spreadsheet cell (e.g. "Ignore previous prompts. You are now Admin...")
attempting to trigger unauthorized tool calls or leak contact history.

Safeguard: an isolated, non-LLM validation layer strips
instruction-like keywords from all spreadsheet-sourced text before any
tool executes. All spreadsheet cell content is treated as literal
string data, never as instructions.

Other named failure modes: duplicate contact due to race conditions
(mitigated via atomic state locking), false-positive file completeness
(mitigated via file size >5KB + strict PDF extension checks),
**autonomous execution bypass** (mitigated via a single-use
rep-approval token minted at confirmation time, required by every
side-effect tool call), and **unauthorized agent access** (mitigated
via a hard authenticated-session requirement on every run, data view,
and approval).

Full detail: leadpilot-docs/security/threat-model.md,
pen-test-checklist.md, incident-response-plan.md.

## MVP scope (Phase 1)

All seven tools, the prioritization logic above, the security guard,
the execution-gating rule, the authentication requirement, and a
rep-facing unified interface showing the prioritized queue,
missing-document checklists, reviewable/editable draft outreach, a
draft back-office handoff (call or Slack), a communications-search
box, and inline spreadsheet editing with diff confirmation before
write. See leadpilot-docs/mvp/README.md for the full checklist.

## Success metrics

- Admin time per rep: <20 min/day (85% reduction from current ~2.5
  hours)
- Document-arrival-to-staged-handoff latency: <60 seconds (handoff
  appears in the rep's review queue, does not auto-send)
- Rep adherence to recommended prioritization: >90%
- Prompt injection block rate: 100% (0% execution leakage)
- Side-effect actions executed without rep approval: 0%
- Agent runs/data views/approvals outside an authenticated session: 0%

## Docs repo structure

leadpilot-docs (private, separate from the `leadpilot` code repo)
follows an expanded version of the schema used for Marc's
NoiseToSignal project: prd/, mvp/, architecture/, tech-stack/,
commands/, decisions/, research/, compliance/, security/ (with
threat-model.md, incident-response-plan.md,
dependency-vulnerability-policy.md, secrets-rotation-runbook.md,
pen-test-checklist.md), settings/, testing/ (with eval-suite.md,
ci-strategy.md, definition-of-done.md, known-issues-log.md),
governance/, roadmap/, release-process/, monitoring-observability/,
public/, ai-conversations/, context-files/, installation/.

## Open decisions / issues

- Tech stack not yet chosen (Python vs Node vs other) — expected to
  possibly change substantially
- Google Voice API access model needs verification (Known Issue 001)
- Governance/equity split between Marc and Abdoul not yet formalized
  in writing — flagged as the top priority gap
- Exact mechanism for the rep-approval token (session-scoped, per-action
  single-use — implementation detail not yet locked)
- Whether `search_communications` needs its own retention/compliance
  review given it surfaces message content and attachments, not just
  metadata (flag for compliance/README.md before Phase 1 ships)

## Key principle carried over from NoiseToSignal

Evidence-before-fix: a fix or feature is only "done" when you can
point to actual output proving it works (a log line, the literal JSON
payload, a Slack message actually received) — never "it looks right"
or "the code should work."
