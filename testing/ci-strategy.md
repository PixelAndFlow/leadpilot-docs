# CI strategy

Status: stub — fill in once the tech stack and hosting are decided.

## What should run automatically on every push (once implemented)

- Lint / type-check
- Unit tests for each tool function (`fetch_all_leads`,
  `get_contact_history`, `verify_drive_contents`,
  `dispatch_slack_handoff`, `search_communications`,
  `update_lead_sheet`, `initiate_backoffice_call`) against mocked API
  responses
- For the four side-effect tools (`dispatch_slack_handoff`,
  `initiate_backoffice_call`, `update_lead_sheet`, and outreach sends),
  a test confirming each rejects execution when no valid rep-approval
  token is present — this is the execution-gating rule from PRD v1.01
  and must never regress
- Case 1, Case 2, Case 4, and Case 5 from testing/eval-suite.md, run
  against a mocked agent (no real API calls, no real Slack messages,
  no real spreadsheet writes)

## What should run before every deploy (not necessarily every push)

- Full eval suite including Case 3 (adversarial input) and Case 6
  (unauthorized access attempt)
- security/pen-test-checklist.md, at least the prompt-injection and
  execution-gating sections

## What stays manual

- Full pen-test-checklist.md sweep — before launch and before major
  releases, not every deploy
- Anything requiring a real Slack send, real phone call, real
  spreadsheet write, or real Google Drive file (mock these in CI;
  verify manually before a real launch)

## Notes

- Never let CI make real Slack posts or real Google API calls — all
  automated tests run against mocks/fixtures.
- Once a CI platform is chosen (GitHub Actions, etc.), add the actual
  workflow file location here and mirror the basic stub already
  placed in the `leadpilot` code repo's `.github/workflows/`.
