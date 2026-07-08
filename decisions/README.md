# decisions/

A log of every significant decision made during LeadPilot's build.
Each entry records what was decided, why, and what alternatives were
considered.

## What belongs here

- All decisions from the log below
- New decisions added during the build
- Decisions that were reversed or changed, with reason why

## Suggested file naming

  decisions-log.md      Single running log of all decisions

## Entry format

  ## Decision [number] — [short title]
  Date: [date]
  Decision: [what was decided]
  Reason: [why this was chosen]
  Alternatives considered: [what else was evaluated and why rejected]
  Status: Active / Superseded by Decision [number]

## Decisions log (current)

| # | Decision | Choice | Rationale |
|---|----------|--------|-----------|
| 001 | Agent name | LeadPilot | Reflects the "autonomous co-pilot for outbound sales" positioning |
| 002 | Ownership | Marc Delsoin, Abdoul Ba (co-owners) | Per PRD v1, July 6, 2026 |
| 003 | Docs repo structure | Adopted the NoiseToSignal docs schema, expanded (governance/, roadmap/, release-process/, monitoring-observability/, public/) | Reuse proven structure; add professional-repo elements identified as gaps before LeadPilot's build starts |
| 004 | Repo separation | Two private repos: `leadpilot` (code) and `leadpilot-docs` (planning/security/decisions) | Security separation and independent access control — see leadpilot-docs/README.md |
| 005 | Tech stack | Deferred | Not yet decided; expected to change substantially — see tech-stack/README.md |
| 006 | Prompt injection defense | Isolated validation layer strips instruction-like keywords from all spreadsheet-sourced text before tool execution | Per PRD 3c blast radius — indirect prompt injection is the identified worst-case failure mode |
| 007 | Duplicate contact prevention | Atomic state locking — run timestamps committed before tool calls are authorized | Per PRD 3c — timing-gap duplicate contact is a named failure mode with brand-damage impact |
| 008 | File completeness validation | Require file size >5KB and strict PDF extension match before counting a document as present | Per PRD 3c — prevents false-positive Slack handoffs on empty/invalid files |

## Open decisions (not yet made — track here as they resolve)

- Whether outreach is auto-sent or requires rep approval before send
  (affects mvp/README.md scope and architecture/dashboard-architecture.md)
- Runtime/stack (tech-stack/README.md)
- Where dedup/run-lock state lives
- CI/CD and hosting choice

## Notes

- Write new decisions at the time they are made, not weeks later.
- This log answers "why does it work this way?"
- When a decision is reversed, mark it Superseded and add the new
  decision with a reference back.
- Decisions 001-002 and 006-008 are sourced directly from
  prd/LeadPilot_PRD_v1.md.
