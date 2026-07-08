# CI strategy

Status: stub — fill in once the tech stack and hosting are decided.

## What should run automatically on every push (once implemented)

- Lint / type-check
- Unit tests for each tool function (`fetch_all_leads`,
  `get_contact_history`, `verify_drive_contents`,
  `dispatch_slack_handoff`) against mocked API responses
- Case 1 and Case 2 from testing/eval-suite.md, run against a mocked
  agent (no real API calls, no real Slack messages)

## What should run before every deploy (not necessarily every push)

- Full eval suite including Case 3 (adversarial input)
- security/pen-test-checklist.md, at least the prompt-injection
  section

## What stays manual

- Full pen-test-checklist.md sweep — before launch and before major
  releases, not every deploy
- Anything requiring a real Slack send or real Google Drive file
  (mock these in CI; verify manually before a real launch)

## Notes

- Never let CI make real Slack posts or real Google API calls — all
  automated tests run against mocks/fixtures.
- Once a CI platform is chosen (GitHub Actions, etc.), add the actual
  workflow file location here and mirror the basic stub already
  placed in the `leadpilot` code repo's `.github/workflows/`.
