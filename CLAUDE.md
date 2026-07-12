# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`leadpilot-docs` is the private planning/documentation repo for
LeadPilot (an AI sales agent). It is a **sibling** repo to the code
repo (`leadpilot`), not a subfolder of it — the two are intentionally
separate (security separation, independent access control, clean git
history; see this repo's `README.md` "Why two repos"). There is no
buildable code here: no commands, no tests, no CI. The unit of work in
this repo is markdown documents and keeping them internally consistent.

## Where things live

| Folder | Contents |
|---|---|
| `prd/` | Product Requirements Documents, one file per version — **never delete old versions**. `prd/README.md` always points at the current file. |
| `decisions/README.md` | The decisions log — why things are built the way they are. Check before changing any established mechanism; log new decisions here rather than silently diverging from a prior one. |
| `mvp/README.md` | The build order (Step 0 accounts/access, Step 1 foundation, Step 2 tools, ...). Steps are sequenced by dependency, not preference. |
| `architecture/README.md`, `architecture/state-schema.md` | Tool map, data flow, system-prompt versions, and the authoritative schema semantics for the state store. |
| `tech-stack/stack-overview.md` | Locked stack and why (Decision 022). |
| `security/threat-model.md`, `security/secrets-rotation-runbook.md`, `security/incident-response-plan.md`, `security/pen-test-checklist.md` | Threats, blast radius, credential rotation, incident response. |
| `compliance/README.md` | Google/Slack API ToS, data handling, GDPR/CCPA for lead PII. |
| `testing/eval-suite.md`, `testing/definition-of-done.md`, `testing/known-issues-log.md`, `testing/ci-strategy.md` | Eval cases, done-bar, open issues, what CI is/isn't allowed to touch. |
| `commands/README.md` | Every terminal command needed to run/build/deploy, and the env var each secret maps to. |
| `ai-conversations/claude/` | Dated logs of significant AI conversations — see header format below. |
| `context-files/` | Reusable context files for restoring AI session state. |
| `governance/`, `release-process/`, `monitoring-observability/`, `research/`, `roadmap/`, `public/`, `settings/`, `installation/` | See root `README.md`'s folder table for what each covers. |

**Never made public as-is:** `security/`, `compliance/`, `governance/`,
`architecture/` — see `public/README.md` for how to safely turn
material from here into public-facing content.

## Conventions to follow when editing

- **Decisions log first.** A change to an established mechanism (access
  model, auth approach, schema, tool behavior) gets a numbered entry in
  `decisions/README.md` — reasoning included, not just the outcome —
  before or alongside updating the docs that implement it. Decisions
  are cross-referenced by number ("supersedes Decision 024", "updates
  Decision 022") rather than silently overwritten.
- **PRD versions are additive.** New PRD content is a new
  `LeadPilot_PRD_vX.XX.md` file; old versions stay in `prd/` and get
  marked superseded in `prd/README.md`, never deleted or edited in place.
- **Propagate changes across every affected doc in the same pass.**
  This repo has failed before by having one doc (e.g. `mvp/README.md`)
  drift out of sync with a decision recorded elsewhere. When a decision
  changes something, grep for the old assumption across `prd/`, `mvp/`,
  `architecture/`, `tech-stack/`, `commands/`, `security/`,
  `compliance/`, and `testing/` rather than assuming one file is
  authoritative.
- **Code-repo drift is expected and should be flagged, not silently
  assumed fixed.** The `leadpilot` code repo lags documentation changes
  (e.g. Decision 026 changed the Google access model in docs before the
  code's `GoogleSheetsConnector` was reworked to match — see that
  repo's `CLAUDE.md` "Known drift to be aware of"). When editing docs
  here, check whether the corresponding code has caught up, and say so
  explicitly if it hasn't.
- **`ai-conversations/claude/` file format:** `YYYY-MM-DD-topic.md`,
  with a header of `# AI Conversation — [topic]`, `Date:`, `Model:`,
  `Summary:`, `Key outputs:`, `Key decisions:`, then `--- transcript
  below ---`. Never paste real API keys, passwords, or credentials into
  a saved transcript.

## Current state (check these first for "what's the latest")

- PRD: `prd/LeadPilot_PRD_v1.05.md` (current) — per-rep OAuth access
  model, per-rep batch run, `fetch_ad_hoc_sheet`.
- Decisions log: through Decision 028.
- Build order: Step 0 (accounts/access) complete; Step 1 (foundation)
  merged to `main` in the code repo; Step 2 (the tools) not started.
