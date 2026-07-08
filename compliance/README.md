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
  Sheets/Drive read access; avoid broad Workspace admin scopes.
- **Data ownership** — the sales org (client), not LeadPilot, likely
  owns the lead data. Document this explicitly before building any
  feature that could be read as LeadPilot retaining or reusing lead
  data across clients.
- **Prompt injection / data leakage** — the security guard described
  in the PRD (3c) is a compliance control as much as a security one:
  it prevents lead data or contact histories from being exposed
  through a manipulated agent response. Cross-reference with
  security/threat-model.md.

## Notes

- This is not a substitute for legal review — have a qualified
  attorney review before handling a real sales org's client data,
  especially financial documents.
- If LeadPilot is sold to multiple sales orgs, revisit this folder for
  multi-tenant data isolation requirements before that becomes real.
