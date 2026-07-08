# release-process/

Deployment runbook and rollback procedure for LeadPilot. Status: stub
— fill in once tech stack and hosting are chosen.

## What belongs here

- Step-by-step deploy procedure
- Pre-deploy checklist (should reference testing/eval-suite.md and,
  before major releases, security/pen-test-checklist.md)
- Rollback procedure — how to revert if a deploy breaks the hourly
  run or produces bad outreach
- Versioning approach for the agent itself (system prompt version,
  tool version)

## Suggested file naming

  deployment-runbook.md     Step-by-step deploy procedure
  rollback-procedure.md     How to revert a bad deploy

## Pre-deploy checklist (draft, stack-agnostic)

- [ ] All three testing/eval-suite.md cases pass
- [ ] No open Issue in testing/known-issues-log.md blocks this release
- [ ] Secrets/credentials confirmed current (not mid-rotation) per
      security/secrets-rotation-runbook.md
- [ ] Change logged in decisions/decisions-log.md if it affects
      prompt, tools, or prioritization logic

## Notes

- Given the hourly-schedule nature of this agent, a bad deploy doesn't
  just risk downtime — it risks a bad prioritized queue or an
  incorrect Slack handoff reaching real people. Treat rollback speed
  as a real requirement once live.
