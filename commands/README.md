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

Stack is undecided (see tech-stack/README.md), so this file is a
placeholder until the runtime is locked. Once decided, fill in:

  # Navigate to project
  cd path/to/leadpilot

  # Install dependencies
  (npm install / pip install -r requirements.txt / etc.)

  # Run one-off (no scheduler) against test data
  (to be defined)

  # Run the eval suite (testing/eval-suite.md cases 1-3)
  (to be defined)

  # Deploy
  (to be defined)

## Environment variables required (known now, independent of stack)

Per the PRD's tool definitions, `leadpilot/.env.example` will need:

  GOOGLE_SHEETS_API_KEY=      or OAuth client credentials
  GOOGLE_VOICE_API_KEY=       or equivalent call-log source credentials
  GOOGLE_DRIVE_API_KEY=       or OAuth client credentials
  SLACK_BOT_TOKEN=            for chat.postMessage
  SLACK_HANDOFF_CHANNEL_IDS=  the 3 back-office stakeholder channels/users

## Notes

- Never commit real credentials — .env stays in .gitignore in the
  code repo; only .env.example (key names, no values) is committed.
- Write every command as if explaining to someone who has never seen
  the project before.
