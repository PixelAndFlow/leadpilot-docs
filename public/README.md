# public/

Conventions for turning material from this private docs repo into
public "building in public" content — without publishing this repo
itself. See decisions/decisions-log.md and the discussion that led
here: this repo stays private; public/ is the curated, rewritten
output layer.

## What belongs here

- Drafts of public posts/updates before they go out (X/Twitter,
  blog, LinkedIn, wherever you're building in public)
- A running "safe to share" version of the decisions log — narrative,
  not raw
- A sanitization checklist to run before anything leaves this repo

## Suggested file naming

  YYYY-MM-DD-post-topic.md     One file per public post/update, drafted here first
  safe-to-share-decisions.md   Public-friendly rewrite of decisions-log.md entries

## Sanitization checklist — run before anything from this repo goes public

- [ ] No account-linked URLs (Google Cloud Console links, Slack
      workspace URLs, etc.)
- [ ] No exact security thresholds, rate limits, or validation
      keyword lists (describe the existence of a safeguard, not its
      exact parameters)
- [ ] No raw AI conversation transcripts (summarize instead)
- [ ] No client/sales-org-identifying information (LeadPilot handles
      real lead and financial data — this is a harder line than
      NoiseToSignal's public-app content)
- [ ] No unresolved governance/equity details
- [ ] Rewritten as narrative/reasoning, not a copy-pasted internal doc

## What's good source material

- decisions/decisions-log.md — the reasoning behind choices is
  exactly what build-in-public audiences respond to
- roadmap/README.md — already written in public-safe outcome language
- research/README.md findings (once populated) — competitive landscape
  commentary is generally safe

## Notes

- When in doubt, run it past the checklist twice. Given LeadPilot
  handles real financial documents (bank statements) for real sales
  orgs, the bar here is higher than for a consumer app like
  NoiseToSignal.
