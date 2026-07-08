# security/

Security decisions, threat modeling, and operational security
practices for LeadPilot. This folder is more heavily developed than
usual at this early stage because the PRD itself is agent-security-
forward (indirect prompt injection is treated as the top blast-radius
risk) — keep it that way.

## What belongs here

- Threat model derived from the PRD's blast radius section
- Credential storage and secrets management (Google/Slack API keys)
- Incident response plan
- Dependency and vulnerability scanning policy
- Secrets rotation runbook
- Pen-test / adversarial-input checklist (ties directly to the PRD's
  eval card case 3)
- Error logging policy — what's logged, what's masked

## Files in this folder

  threat-model.md                Full threat model, expanded from PRD 3c
  incident-response-plan.md      What happens when something goes wrong
  dependency-vulnerability-policy.md   Scanning cadence, CVE review process
  secrets-rotation-runbook.md    How and when API keys/tokens get rotated
  pen-test-checklist.md          Adversarial input test checklist

## Public-safe counterpart

A public-facing responsible-disclosure policy (SECURITY.md) lives in
the `leadpilot` code repo, not here — that one tells outside
researchers how to report a vulnerability. This folder holds the
internal detail; the code repo's SECURITY.md stays high-level and
contact-info-only.

## Notes

- Update this folder every time the eval card (prd/ section 3d,
  testing/eval-suite.md) gains a new adversarial case — the threat
  model and the eval suite should never drift apart.
- All decisions here should also appear in decisions/decisions-log.md
  when they're first made.
