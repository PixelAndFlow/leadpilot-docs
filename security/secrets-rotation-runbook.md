# Secrets rotation runbook

Status: stub — fill in exact commands once the hosting platform is
chosen (tech-stack/README.md, commands/README.md).

## Secrets in scope

- `GOOGLE_OAUTH_CLIENT_ID` / `GOOGLE_OAUTH_CLIENT_SECRET` — one shared
  OAuth client covering Sheets, Drive, AND Gmail (Decision 026,
  reversed 2026-07-11 — **supersedes** the service-account plan below).
  Rotation here means generating new client credentials in Google
  Cloud Console; existing reps' stored refresh tokens are unaffected
  by rotating the client secret itself
- `GOOGLE_PICKER_API_KEY` — client-side key for the Google Picker
  widget (Decision 026). Unlike every other secret in this list, this
  one can't actually be kept confidential: the Picker widget runs in
  the rep's own browser, so the key has to ship inside the page's
  JavaScript, visible to anyone who opens dev tools, views page
  source, or inspects network requests. Rotation and confidentiality
  don't do the protective work here — restriction does. Created
  2026-07-11 (`LeadPilot_Google_Picker`) with two restrictions instead:
  API restriction limited to the Google Picker API only (so a copied
  key can't be used to call any other Google API on this project), and
  application restriction limited to specific website referrers
  (`http://localhost:8000/*` for now, the real domain once deployed —
  Google rejects calls using this key from any other origin). Still
  tracked here and rotatable independently, but "rotation" matters less
  for this key than getting the restrictions right
- **Per-rep OAuth refresh tokens** (new, Decision 026) — not a single
  static secret. Each rep's refresh token is stored encrypted in
  Postgres (`rep_google_credentials`, see architecture/state-schema.md),
  not an env var. "Rotation" for an individual rep means that rep
  disconnecting and re-consenting; a compromise of the encryption
  key protecting this column is the actual incident-scale rotation
  event (re-encrypt all stored tokens, force every rep to reconnect)
- ~~`GOOGLE_SERVICE_ACCOUNT_KEY_PATH`~~ — superseded by Decision 026.
  The Step 1 code merged to `main` still uses this for local dev
  against the not-yet-reworked `GoogleSheetsConnector` (Decision 024),
  so it isn't removed from `.env.example` yet — see commands/README.md.
  Once Step 2 reworks the connector for per-rep OAuth, this key and
  its rotation procedure (generating a new service-account key in
  Google Cloud Console) go away entirely
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

**Per-rep refresh token compromise (new, Decision 026):** revoke that
rep's Google OAuth grant from the Google Account permissions page (or
the Workspace admin console), delete the stored row in
`rep_google_credentials`, and have the rep reconnect via the dashboard
(re-consent + re-pick sheets via the Picker). This is a per-rep
procedure, not a global credential swap — other reps' access is
unaffected.

## Notes

- Never commit real values to either repo — `leadpilot/.env.example`
  contains key names only.
- Log every rotation (scheduled or incident-triggered) in
  decisions/decisions-log.md or a dedicated rotation log if this
  becomes frequent enough to warrant one.
