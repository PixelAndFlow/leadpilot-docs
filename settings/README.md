# settings/

Design documentation for LeadPilot's configuration surface — what's
configurable, what each setting controls, and how it's stored. For an
agent product this covers both rep-facing dashboard settings and
operator-level agent configuration (schedule, thresholds).

## What belongs here

- Rep-facing dashboard settings (if any) — e.g. default queue sort,
  notification preferences
- Operator-level agent configuration — run frequency, prioritization
  thresholds (e.g. the "24 hours" window for Rank 1), file size
  threshold for document validation, the 3 back-office stakeholder
  accounts for Slack handoff
- How settings are stored and how config changes take effect (restart
  required vs. live reload)

## Suggested file naming

  settings-phase1.md      All settings at Phase 1 launch
  settings-roadmap.md     Future settings for later phases

## Known configurable values from the PRD (confirm which become
user-editable settings vs. hard-coded constants)

| Value | PRD source | Editable or fixed? |
|---|---|---|
| Rank 1 recency window (24 hours) | System prompt v0, prioritization logic | TBD |
| File size validation threshold (5KB) | 3c, file completeness safeguard | Recommend fixed — this is a security control, not a preference |
| Back-office stakeholder accounts (3 named) | System prompt v0, dispatch step | Recommend fixed/hard-coded, not derived from any editable or user-influenced input — see security/threat-model.md scope-expansion risk |
| Run schedule (hourly) | Section 2, "persistent hourly schedule" | TBD |

## Notes

- Be deliberate about which of the above are true "settings" (safe to
  expose as editable) vs. security-relevant constants that should stay
  hard-coded regardless of UI convenience. The Slack stakeholder list
  is the clearest example of the latter.
