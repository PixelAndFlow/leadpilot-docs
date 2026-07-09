# governance/

Ownership, roles, and decision-making structure for LeadPilot. This
folder exists because working with a partner (Abdoul) makes informal
"we both just know how this works" arrangements risky — the goal is
to have this written down before it's ever actually in dispute, not
after.

**This folder is a starting scaffold, not a legal document.** Nothing
here should be treated as binding until reviewed by a qualified
attorney. Writing it down now, even informally, is still better than
having nothing.

## What belongs here

- Ownership / equity split (if applicable)
- IP assignment — who owns the code, the agent's prompts/logic, and
  the LeadPilot name if the partnership ends or changes
- Roles and responsibilities between Marc and Abdoul
- Decision-making authority — who has final say on what (technical
  architecture, product scope, security posture, business decisions)
- What happens if one partner wants to leave, or the partnership
  dissolves

## Suggested file naming

  ownership-and-roles.md       Current split of responsibility and equity
  decision-rights.md            Who decides what, and how ties are broken
  partnership-agreement-status.md   Tracks whether a real legal agreement exists yet

## Current state (fill in / confirm with Abdoul)

- **Owners:** Marc Delsoin, Abdoul Ba (per PRD v1, July 6, 2026)
- **Equity split:** Equal — 50/50 between Marc and Abdoul (confirmed
  2026-07-08). Still worth a real written agreement (see
  partnership-agreement-status.md) — an equal split doesn't on its own
  answer what happens to the code, the name, or client data if one
  partner leaves or the partnership ends.
- **Roles:** Not yet divided in writing
- **Decision rights:** Either partner may fork or use the work as they
  see fit (confirmed 2026-07-08) — i.e. no exclusive veto held by
  either side. Recommend still writing down what "use it as they see
  fit" means in practice before it's tested by an actual disagreement
  (e.g. can one partner take it commercial solo, what happens to the
  LeadPilot name, who owns client data collected under one partner's
  version).

## Notes

- This is the single highest-priority gap flagged before LeadPilot's
  build starts — informal partnerships are the most common source of
  damage when things go well (who owns the upside) and when they
  don't (who's responsible, who can walk away).
- Revisit and formalize before the first paying client or any
  external funding conversation.
