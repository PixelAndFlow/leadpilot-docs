# leadpilot-docs

Documentation, planning, research, and AI conversation logs for LeadPilot.
This is a private repository, separate from the `leadpilot` code repository.
Do not make this repo public.

LeadPilot is an AI agent for B2B sales and business development teams. It
orchestrates lead triage and multi-channel communication across Google
Workspace (Sheets, Drive) and Slack, running on a persistent hourly
schedule. It parses disparate lead sheets, audits contact history to
prevent duplicate outreach, verifies document completeness for deal
handoff, and pushes structured execution logs to a rep-facing dashboard.

Owners: Marc Delsoin, Abdoul Ba

## Why two repos

`leadpilot` (code) and `leadpilot-docs` (this repo) are intentionally
separate, both private:

- Security separation — this repo holds threat models, security
  thresholds, compliance timelines, and API/account details that
  shouldn't ship inside the code repo's git history, which may be
  shared more widely over time (contractors, CI systems, eventual
  public release).
- Independent access control — collaborators can be added to the code
  repo without automatically granting access to planning/security docs.
- Clean history — docs churn (rewritten plans, versioned context files)
  doesn't pollute code review history.

Both repos are private on GitHub. Add collaborators individually.

## Folder overview

| Folder | Contents |
|--------|----------|
| prd/ | Product Requirements Document(s) for each build phase |
| mvp/ | MVP scope, value props, and feature checklists |
| architecture/ | Agent orchestration flow, tool map, data flow, system prompt versions |
| tech-stack/ | Technologies used/considered, versions, and why (currently undecided) |
| commands/ | Every terminal command needed to run, build, or deploy |
| decisions/ | Log of every significant decision made and why |
| research/ | Competitive analysis, API research, platform comparisons |
| compliance/ | Google API / Slack API ToS, data handling, GDPR/CCPA for lead PII |
| security/ | Threat model, blast radius, encryption, incident response, dependency policy |
| testing/ | Manual test plans, eval card / regression suite, CI strategy, definition of done |
| settings/ | Settings/config panel design, what each setting does |
| governance/ | Ownership, roles, decision rights, partnership terms (needs legal review) |
| roadmap/ | Public-facing phase roadmap |
| release-process/ | Deployment runbook, rollback procedure |
| monitoring-observability/ | Logging, alerting, uptime strategy |
| public/ | Build-in-public conventions — turning private decisions into public posts |
| ai-conversations/ | Saved AI chat logs organized by model |
| context-files/ | Context files used to restore AI session state, and the reusable docs-schema template |
| installation/ | Setup guides for local dev dependencies |

## Important

This repo is private. Several folders (security/, compliance/,
governance/, architecture/) contain information that should never be
made public as-is. See `public/README.md` for how to turn material
from this repo into public-facing content safely.
