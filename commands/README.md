# commands/

Every command needed to work with LeadPilot — setup, development,
scheduling, deployment, and debugging. Written so you or Abdoul can
pick this up after months away and get running in minutes.

## What belongs here

- Complete setup instructions from scratch on a new machine
- Commands to run the agent loop locally (one-off, not on schedule)
- Commands to test against the eval card without hitting live APIs
- Deployment and scheduler setup commands
- Debugging commands for common problems

## Suggested file naming

  commands.md                  Master command reference

## Status

Stack locked (Decision 022 — Python, Claude Agent SDK, FastAPI,
Postgres/Neon, Render). Full detail in tech-stack/stack-overview.md.

  # Navigate to project
  cd path/to/leadpilot

  # Install dependencies
  pip install -r requirements.txt

  # Run the batch agent loop once, locally, against test data
  # (no scheduler involved — this is what the Render Cron Job calls)
  python -m leadpilot.run_batch

  # Run the dashboard/API locally
  uvicorn leadpilot.app:app --reload

  # Run the eval suite (testing/eval-suite.md, all 10 cases)
  pytest tests/eval_suite/

  # Deploy
  # Push to main — Render auto-deploys both the Web Service and the
  # Cron Job from the same repo

## Environment variables required

Per PRD v1.04's tool definitions and Decision 022's stack,
`leadpilot/.env.example` will need:

  ANTHROPIC_API_KEY=          Claude Agent SDK / tool-calling loop
  DATABASE_URL=                Neon Postgres connection string — the
                               shared state store (contact history +
                               approval gate + dedup/run-lock, see
                               architecture/state-schema.md)
  GOOGLE_OAUTH_CLIENT_ID=      Sheets, Drive, and Gmail scopes — Sheets
  GOOGLE_OAUTH_CLIENT_SECRET=  needs read AND write (update_lead_sheet);
                               Gmail used for send_lead_email and feeds
                               search_communications
  TWILIO_ACCOUNT_SID=          send_lead_text and the SMS half of
  TWILIO_AUTH_TOKEN=           search_communications — no Google Voice
  TWILIO_FROM_NUMBER=          credential exists or is used anywhere
                               (testing/known-issues-log.md Issue 001)
  SLACK_BOT_TOKEN=             for chat.postMessage (dispatch_slack_handoff)
  SLACK_HANDOFF_CHANNEL_IDS=   the 3 back-office stakeholder channels/users
  REP_AUTH_SESSION_SECRET=     signs/validates authenticated rep sessions
                               (Decision 013 access-control requirement)

Note what's deliberately *not* here: there is no
`APPROVAL_TOKEN_SECRET` or equivalent. Decision 021 resolved the
rep-approval mechanism as a conditional database update on the
contact-history log's own `stage` column — no signed or opaque token
object exists to configure.

## Notes

- Never commit real credentials — .env stays in .gitignore in the
  code repo; only .env.example (key names, no values) is committed.
- Write every command as if explaining to someone who has never seen
  the project before.
