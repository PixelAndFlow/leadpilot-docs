# Secrets rotation runbook

Status: stub — fill in exact commands once the hosting platform is
chosen (tech-stack/README.md, commands/README.md).

## Secrets in scope

- `GOOGLE_SHEETS_API_KEY` (or OAuth client credentials)
- `GOOGLE_VOICE_API_KEY` (or equivalent — pending research/README.md)
- `GOOGLE_DRIVE_API_KEY` (or OAuth client credentials)
- `SLACK_BOT_TOKEN`

## Rotation triggers

- Scheduled rotation (cadence TBD — suggest every 90 days once live)
- Suspected compromise (trigger incident-response-plan.md immediately,
  don't wait for the scheduled window)
- Personnel change (if anyone with credential access leaves the
  project)

## Rotation procedure (draft)

1. Generate new credential in the relevant console (Google Cloud
   Console / Slack App management)
2. Add new credential to the hosting platform's environment variables
   alongside the old one
3. Deploy/restart so the agent picks up the new credential
4. Verify the agent successfully calls all four tools with the new
   credential (run the eval suite smoke case)
5. Revoke the old credential
6. Confirm no errors in the next scheduled run

## Notes

- Never commit real values to either repo — `leadpilot/.env.example`
  contains key names only.
- Log every rotation (scheduled or incident-triggered) in
  decisions/decisions-log.md or a dedicated rotation log if this
  becomes frequent enough to warrant one.
