# research/

Competitive analysis, API research, platform comparisons, and market
context informing LeadPilot's design and technical decisions.

## What belongs here

- Competitive analysis of existing tools (traditional CRMs — Salesforce,
  HubSpot; sales engagement platforms — Outreach, Salesloft; AI SDR
  tools)
- Google Workspace API research (Sheets, Voice, Drive) — quota,
  auth model, rate limits
- Slack Web API research — rate limits, app scopes needed for
  `chat.postMessage`
- Hosting/scheduler platform comparisons for an hourly-run agent
- Market context: outbound sales workflows, why reps abandon CRM
  discipline, prompt-injection risk in agent products generally

## Suggested file naming

  competitive-analysis.md      CRMs, sales engagement platforms, AI SDR tools
  google-api-research.md       Sheets/Voice/Drive auth, quota, rate limits
  slack-api-research.md        Scopes, rate limits, message formatting
  hosting-scheduler-comparison.md  Platform evaluations for hourly runs
  market-notes.md               Outbound sales workflow context

## To research before Phase 1 build starts

- Google Voice API access model — confirm whether call-log data is
  actually available via a documented API, or whether this requires
  a different integration (this is a named risk: the PRD assumes
  `GET /v1/communications/logs` exists — verify before committing to
  this architecture)
- Slack app scopes required for `chat.postMessage` to specific users
  vs. channels
- Whether existing AI SDR / sales agent tools already solve this
  problem well enough to change the build-vs-buy calculus

## Notes

- This folder is currently empty of findings — populate before
  locking the tech stack, since the Google Voice API question above
  could materially change the architecture.
