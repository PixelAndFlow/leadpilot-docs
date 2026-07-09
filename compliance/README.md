# compliance/

Legal and policy requirements for LeadPilot. Documents what is
required, what has been addressed, and what still needs to be done
before deploying against a real sales org's data.

## What belongs here

- Google Workspace API Terms of Service compliance (Sheets, Drive)
- Google Voice API compliance (data source/access model still needs
  research — see research/README.md)
- Slack API Terms of Service compliance
- GDPR / CCPA requirements for lead PII (names, phone numbers, emails,
  financial documents like bank statements)
- Data retention and deletion requirements for prospect data and
  uploaded financial documents
- Internal data handling agreement with the sales org LeadPilot serves
  (who owns the lead data, what LeadPilot is allowed to do with it)

## Suggested file naming

  google-api-compliance.md       Sheets/Drive/Voice ToS mapped to design
  slack-api-compliance.md        Slack ToS and app scope justification
  data-handling-agreement.md     What LeadPilot may do with client lead data
  gdpr-ccpa-requirements.md      EU/CA compliance for PII and financial docs
  data-retention.md              What is stored, how long, deletion process

## Critical considerations — must be addressed before handling real leads

- **Financial document handling** — bank statements are sensitive
  financial PII. Confirm retention limits, encryption at rest, and
  who (beyond the 3 back-office stakeholders) can access verified
  files.
- **Google API scopes** — request the minimum scopes needed for
  Sheets/Drive read/write access (write access is new as of PRD
  v1.01's `update_lead_sheet` tool); avoid broad Workspace admin
  scopes.
- **Data ownership** — the sales org (client), not LeadPilot, likely
  owns the lead data. Document this explicitly before building any
  feature that could be read as LeadPilot retaining or reusing lead
  data across clients.
- **Prompt injection / data leakage** — the security guard described
  in the PRD (3c) is a compliance control as much as a security one:
  it prevents lead data or contact histories from being exposed
  through a manipulated agent response. Cross-reference with
  security/threat-model.md.
- **Communications-search data surface (new, PRD v1.01)** —
  `search_communications` reads actual email/text content and
  attachments, not just contact metadata. This needs its own retention
  and access review before use against real client data — see
  testing/known-issues-log.md Issue 004 and
  security/threat-model.md's note on this tool.
- **Authenticated access (new, PRD v1.01)** — only authenticated,
  pre-authorized rep accounts may trigger the agent, view lead data,
  or approve staged actions (decisions/README.md Decision 013). This
  is also a data-handling-agreement consideration: document what
  access logging is kept and for how long.

## Rep-approval gating is not a compliance substitute (added 2026-07-08)

The rep-approval gate (every outreach send requires explicit rep
sign-off, PRD 3a Execution-gating rule) closes the "did a human
actually decide to send this" gap and meaningfully reduces
bulk/autodialer-style liability — a rep individually reviewing and
triggering each message is a different risk profile than a system
blasting messages unattended. But it does not by itself satisfy any
of the following, which remain outstanding regardless of who clicks
approve:

- **TCPA (calls/texts)** — still need documented prior consent to
  contact a lead's phone number for calls/texts, and a compliant
  opt-out mechanism (e.g. honoring STOP on texts). Rep approval
  doesn't establish that consent exists.
- **CAN-SPAM (email)** — every commercial email needs an unsubscribe
  mechanism, honored opt-outs, and accurate sender/physical-address
  info, whether a human reviewed it or not.
- **Do-not-call list scrubbing** — needs to happen before a number is
  ever surfaced for a call or text, independent of approval.
- **Communications-search exposure (Issue 004)** — `search_communications`
  is a read operation; the approval gate governs sends, not reads, so
  this needs its own review regardless.
- **Data ownership / retention** — unaffected by who approves what;
  still needs the data-handling-agreement work listed above.

## Notes

- This is not a substitute for legal review — have a qualified
  attorney review before handling a real sales org's client data,
  especially financial documents.
- If LeadPilot is sold to multiple sales orgs, revisit this folder for
  multi-tenant data isolation requirements before that becomes real.
