# Incident response plan

Status: draft skeleton — fill in with real contacts and procedures
before LeadPilot handles real sales org data. This is intentionally a
stub so the shape exists even before it's fully populated.

## What counts as an incident

- A prompt injection attempt succeeds in triggering an unauthorized
  tool call (real Slack message sent, real data exposed)
- Duplicate contact occurs despite the atomic locking safeguard
- A credential (Google API key, Slack bot token) is exposed or
  suspected compromised
- Lead PII or financial documents (bank statements) are exposed to
  the wrong party
- The agent is unavailable during a scheduled run for longer than
  [threshold TBD]

## Response steps (draft — confirm with Abdoul before treating as final)

1. **Contain** — disable the affected tool or pause the scheduler
   (specific command TBD once tech stack is chosen — see commands/).
2. **Assess** — determine what data or action was affected using the
   error/sync logs (see monitoring-observability/README.md).
3. **Notify** — who gets told, and how fast. For a credential leak:
   rotate immediately (see secrets-rotation-runbook.md) before
   notifying. For a data exposure: notify the affected sales org
   per the data handling agreement (compliance/).
4. **Fix** — patch the validation layer, tighten the eval suite with
   a new regression case, and log the incident and fix in
   decisions/decisions-log.md.
5. **Review** — add the failure mode to security/threat-model.md if
   it wasn't already covered, and add the reproducing input to
   testing/eval-suite.md as a new adversarial case.

## Open items

- Named on-call / responsible party (currently Marc and Abdoul only —
  fine for now, revisit if the team grows)
- Notification SLA to affected sales orgs (compliance-driven — needs
  legal input)
- Whether an incident requires pausing the entire hourly run or just
  the affected tool
