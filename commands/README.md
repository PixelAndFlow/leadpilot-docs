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

  # Run the batch agent loop once, locally, against real (or test) data
  # (no scheduler involved — this is what the Render Cron Job calls)
  python -m leadpilot.agent_run

  # Run the dashboard/API locally
  uvicorn leadpilot.app:app --reload

  # Run the structural eval-suite regression tests (9 of 11 cases —
  # Cases 1/2 need a real model and live via scripts/run_evals.py
  # instead, see below)
  pytest tests/eval_suite/

  # Run the full eval suite live against a real model (all 11 cases,
  # Google faked, staging-only — nothing external can fire). Needs
  # ANTHROPIC_API_KEY in .env.local.
  python scripts/run_evals.py

  # Deploy (Decision 022/037, render.yaml added 2026-07-15)
  # First time only — this repo isn't connected to Render yet:
  #   1. Render dashboard -> New -> Blueprint -> connect this repo.
  #      Render reads render.yaml and creates both services
  #      (leadpilot-web, leadpilot-agent-run) automatically.
  #   2. Set every env var marked `sync: false` in render.yaml by hand
  #      in each service's dashboard — real secrets are never in the
  #      committed file. See "Environment variables required" below
  #      and secrets-rotation-runbook.md.
  #   3. Trigger the first deploy from the dashboard.
  # After that first connection: push to main -> Render auto-deploys
  # both services from the same repo, no manual step needed.

## Environment variables required

Per PRD v1.05's tool definitions and Decision 022's stack,
`leadpilot/.env.example` will need:

  ANTHROPIC_API_KEY=          Claude Agent SDK / tool-calling loop
  DATABASE_URL=                Neon Postgres connection string — the
                               shared state store (contact history +
                               approval gate + dedup/run-lock +
                               per-rep Google credentials, see
                               architecture/state-schema.md)
  GOOGLE_OAUTH_CLIENT_ID=      Sheets, Drive, AND Gmail — one per-rep
  GOOGLE_OAUTH_CLIENT_SECRET=  OAuth client (Decision 026, reversed
                               2026-07-11, supersedes Decision 024's
                               service-account plan). Each rep does a
                               one-time consent and picks specific
                               sheets/folders via the Google Picker —
                               nothing is pre-shared to a standing
                               identity. Sheets use drive.file;
                               verify_drive_contents needs drive.readonly
                               too as of Decision 033 (drive.file's
                               per-item grant doesn't extend to a
                               folder's contents — see
                               compliance/README.md for the Google
                               restricted-scope review this triggers).
                               Gmail consent (send_lead_email,
                               Step 2) uses the same client. LeadPilot
                               stores each rep's refresh token in
                               Postgres (rep_google_credentials, see
                               architecture/state-schema.md), not as a
                               single env var — there is no more
                               GOOGLE_SERVICE_ACCOUNT_KEY_PATH or
                               GOOGLE_SHEETS_SOURCES; access is
                               per-rep and Picker-driven, not a static
                               admin-configured list
  GOOGLE_PICKER_API_KEY=       Client-side key for the Google Picker
                               widget (Decision 026) — lets a rep
                               select which sheets/folders to grant,
                               separate from the OAuth client secret
  TWILIO_ACCOUNT_SID=          send_lead_text and the SMS half of
  TWILIO_AUTH_TOKEN=           search_communications — no Google Voice
  TWILIO_FROM_NUMBER=          credential exists or is used anywhere
                               (testing/known-issues-log.md Issue 001)
  SLACK_BOT_TOKEN=             for chat.postMessage (dispatch_slack_handoff)
  SLACK_HANDOFF_CHANNEL_IDS=   the 3 back-office stakeholder channels/users
  REP_AUTH_SESSION_SECRET=     signs/validates authenticated rep sessions
                               (Decision 013 access-control requirement)

Note: the Step 1 code currently merged to `main` still reads
`GOOGLE_SERVICE_ACCOUNT_KEY_PATH`/`GOOGLE_SHEETS_SOURCES` (Decision
024's now-superseded model) — those variables still work for local
dev against the existing `GoogleSheetsConnector` until Step 2 reworks
it for Decision 026. Don't remove them from `.env.example` until that
rework lands.

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
