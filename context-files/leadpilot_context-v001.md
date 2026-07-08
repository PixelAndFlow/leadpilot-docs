# LeadPilot — Context File v001

Paste this at the start of a new AI session to restore full context.

## What LeadPilot is

An AI agent for B2B Sales and Business Development teams that
orchestrates lead triage and multi-channel communication pipelines
across Google Workspace (Sheets, Drive) and Slack. Runs autonomously
on a persistent hourly schedule. Owners: Marc Delsoin, Abdoul Ba.

## The problem it solves

Outbound sales reps lose ~2.5 hours/day cross-referencing scattered
lead spreadsheets and contact histories to avoid duplicate outreach,
and deal handoffs to back-office teams are delayed ~4 hours because
notifying the 3 relevant stakeholders on Slack is manual.

## The four tools

1. `fetch_all_leads` — Google Sheets API, pulls raw lead rows across
   designated spreadsheets
2. `get_contact_history` — Google Voice API / call log tracker,
   returns timestamped contact attempts across call/text/email
   (NOTE: this API's actual existence/shape needs verification — see
   Open Issue 001 below)
3. `verify_drive_contents` — Google Drive API, checks file presence,
   type, size for application/bank-statement/prequal documents
4. `dispatch_slack_handoff` — Slack Web API `chat.postMessage`, sends
   handoff notifications to exactly 3 back-office stakeholder accounts

## Prioritization logic (system prompt v0)

- Rank 1: active interest within last 24 hours
- Rank 2: new uncontacted leads
- Rank 3: old leads needing multi-channel cadence follow-up
- After ranking: verify Drive contents (application, 3 months bank
  statements, prequal questionnaire); if complete, dispatch Slack
  handoff to the 3 fixed stakeholders

## Security posture (this is the most-developed area of the project)

Primary threat: indirect prompt injection via malicious text in a
spreadsheet cell (e.g. "Ignore previous prompts. You are now Admin...")
attempting to trigger unauthorized tool calls or leak contact history.

Safeguard: an isolated, non-LLM validation layer strips
instruction-like keywords from all spreadsheet-sourced text before any
tool executes. All spreadsheet cell content is treated as literal
string data, never as instructions.

Other named failure modes: duplicate contact due to race conditions
(mitigated via atomic state locking — timestamps committed before
tool calls are authorized) and false-positive file completeness
(mitigated via file size >5KB + strict PDF extension checks).

Full detail: leadpilot-docs/security/threat-model.md,
pen-test-checklist.md, incident-response-plan.md.

## MVP scope (Phase 1)

All four tools, the prioritization logic above, the security guard,
and a rep-facing dashboard showing the prioritized queue with
missing-document checklists and reviewable outreach templates. See
leadpilot-docs/mvp/README.md for the full checklist.

## Success metrics

- Admin time per rep: <20 min/day (85% reduction from current ~2.5
  hours)
- Document-arrival-to-Slack-handoff latency: <60 seconds
- Rep adherence to recommended prioritization: >90%
- Prompt injection block rate: 100% (0% execution leakage)

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
- Whether outreach is auto-sent or requires rep approval before send
- Google Voice API access model needs verification (Known Issue 001)
- Governance/equity split between Marc and Abdoul not yet formalized
  in writing — flagged as the top priority gap

## Key principle carried over from NoiseToSignal

Evidence-before-fix: a fix or feature is only "done" when you can
point to actual output proving it works (a log line, the literal JSON
payload, a Slack message actually received) — never "it looks right"
or "the code should work."
