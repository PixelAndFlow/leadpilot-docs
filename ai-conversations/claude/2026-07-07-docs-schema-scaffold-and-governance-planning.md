# AI Conversation — LeadPilot docs schema scaffold and governance planning

Date: 2026-07-07
Model: Claude (Sonnet 5, Cowork mode)
Summary: Marc asked to reuse the NoiseToSignal docs schema for a new
project, LeadPilot (an AI sales-lead agent, co-owned with Abdoul Ba).
Along the way, discussed whether to make docs public (decided: keep
private, add a public/ folder for curated build-in-public content),
and whether to merge the docs repo into the code repo for partner
convenience (decided: keep two separate private repos — least
security exposure, collaborator access covers the convenience need).
Marc then asked what a "professional" repo might be missing even
without direct experience — surfaced governance/partnership docs,
formalized security sub-docs, formalized testing/CI docs, a public
roadmap, release process, and monitoring as gaps. Marc provided the
LeadPilot PRD v1 (agent build spec: 4 tools, system prompt v0, blast
radius/failure modes, eval card with 3 test cases).

Key outputs:
- Full leadpilot-docs schema scaffolded (19 folders): the original 14
  from noisetosignal-docs plus governance/, roadmap/, release-process/,
  monitoring-observability/, public/
- leadpilot code repo skeleton with professional files (README,
  LICENSE, CONTRIBUTING, CHANGELOG, issue/PR templates, CODEOWNERS,
  CI stub, .env.example)
- noisetosignal-docs upgraded in place to match the new standard
- Two context files written: one documenting the NoiseToSignal
  upgrade, one a de-branded reusable version of the schema itself for
  future projects

Key decisions:
- Docs repo stays private; a curated public/ folder handles
  build-in-public content instead of publishing the raw repo
- Two separate private repos (leadpilot, leadpilot-docs), not merged
  and not a submodule — partner (Abdoul) gets collaborator access to
  both
- Governance/partnership documentation flagged as the highest-priority
  gap given a two-person ownership structure with no written
  agreement yet
- Tech stack intentionally left undecided/stubbed — PRD and security
  planning proceed stack-agnostic

--- transcript below ---

(See the live conversation history for full transcript — this file is
a summary header only. Paste the full transcript here if you want a
complete standalone record.)
