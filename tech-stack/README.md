# tech-stack/

Every technology used in LeadPilot — what it is, why it was chosen,
what version, and any important gotchas.

## Status: undecided

Marc and Abdoul have not locked the runtime/stack yet, and expect it
may change substantially as the PRD is refined. This file is a
placeholder — fill in `stack-overview.md` once decisions are made,
and log the reasoning in `decisions/decisions-log.md` the same way
NoiseToSignal's stack choices were logged.

## What belongs here (once decided)

- Master list of all tools, libraries, and frameworks
- Version of each dependency when added
- Why each was chosen over alternatives
- Known limitations, gotchas, configuration decisions
- What was considered and rejected, and why

## Suggested file naming

  stack-overview.md      Master list of all technologies

## Options on the table (not yet decided)

| Option | Notes |
|---|---|
| Python agent loop (LangChain or custom) | Common for tool-calling agents with scheduled jobs; large ecosystem for Google API + Slack SDKs |
| Node.js agent (Claude Agent SDK or custom) | Consistent with Marc's NoiseToSignal/JS background |
| Orchestration platform (n8n, Make, etc.) | Faster to prototype the hourly-run + tool-call shape, less control over the security guard layer described in the PRD |

## Things to decide before locking this in

- Scheduler: cron, a hosted scheduler, or a workflow platform's
  built-in trigger
- Where the state store lives (flat file, SQLite, Postgres, Redis) —
  it now does three jobs, not just one: run-lock/dedup state, the
  contact-history log (architecture/state-schema.md), and rep-approval
  enforcement (Decision 021). Whatever's chosen must support an atomic
  conditional update (`UPDATE ... WHERE stage = X`, checking
  affected-row-count) — standard in SQLite/Postgres, so this shouldn't
  narrow the options, but test it under concurrent access once chosen
- How the rep-facing dashboard is served (static site + API, or
  built into the orchestration platform's UI if using one)
- Hosting — Render/Railway/Fly.io style host vs. serverless scheduled
  function

## Notes

- Do not let stack indecision block the PRD, MVP scope, or security
  planning — those are stack-agnostic and can proceed now.
- Once the stack is chosen, update architecture/system-overview.md
  and commands/commands.md together.
