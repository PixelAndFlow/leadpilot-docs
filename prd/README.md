# prd/

Product Requirements Documents for every phase of LeadPilot. The PRD
defines the problem, users, agent design, tools, blast radius, and
evaluation criteria before any code is written.

## What belongs here

- Filled-out PRD documents for each build phase
- One file per phase/major revision, versioned clearly
- Both current and all previous versions (never delete old versions)

## Suggested file naming

  LeadPilot_PRD_v1.md        Initial agent build PRD
  LeadPilot_PRD_v2.md        Revised after scoping changes

## Current file

  LeadPilot_PRD_v1.01.md — Agent Build PRD, dated 2026-07-07
                           Owners: Marc Delsoin, Abdoul Ba
                           Revises v1 with rep-approval gating, the
                           unified spreadsheet-edit interface, the
                           communications-search tool, and access
                           control. Still a working draft.

## Previous versions

  LeadPilot_PRD_v1.md   — Agent Build PRD, dated 2026-07-06
                           Superseded by v1.01 — kept unedited as the
                           historical starting point. Raw change notes
                           that produced v1.01 are in
                           `LeadPilot_PRD_v1-changed to make.md`.

## Notes

- The PRD is the source of truth for what is being built. If a tool,
  behavior, or output format is not in the PRD, it should not be in
  the code without a corresponding decision entry explaining the change.
- The system prompt version (3b) and eval card (3d) in the PRD are
  living documents — bump the version number in architecture/ and
  testing/ when either changes, and log why in decisions/.
