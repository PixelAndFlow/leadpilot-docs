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

  LeadPilot_PRD_v1.06.md — Agent Build PRD, dated 2026-07-12
                           Owners: Marc Delsoin, Abdoul Ba
                           One change from v1.05: `verify_drive_contents`
                           now authenticates via `drive.readonly`, not
                           `drive.file` — `drive.file`'s per-item Picker
                           grant doesn't extend to a folder's contents,
                           confirmed live while building Step 2 (see
                           decisions/README.md Decision 033, and
                           compliance/README.md for the Google
                           restricted-scope verification consequence
                           this triggers). Still a working draft.

## Previous versions

  LeadPilot_PRD_v1.05.md — Agent Build PRD, dated 2026-07-11
                           Superseded by v1.06. Reverses the Google
                           Sheets/Drive access model from a shared
                           service account to per-rep OAuth (`drive.file`
                           + Google Picker), makes the hourly batch run
                           per-rep, and adds `fetch_ad_hoc_sheet` for
                           on-demand rep-directed sheet access.
  LeadPilot_PRD_v1.04.md — Agent Build PRD, dated 2026-07-08
                           Superseded by v1.05. Retires
                           `initiate_backoffice_call`, redefines
                           `get_contact_history` against a self-owned
                           log, adds `log_call_outcome`.
  LeadPilot_PRD_v1.03.md — Superseded by v1.04. Redefines
                           `initiate_lead_call` as a clipboard handoff.
  LeadPilot_PRD_v1.02.md — Superseded by v1.03. Adds the outreach
                           tools and the `LeadSourceConnector`
                           interface.
  LeadPilot_PRD_v1.01.md — Superseded by v1.02. Adds rep-approval
                           gating, the unified spreadsheet-edit
                           interface, the communications-search tool,
                           and access control.
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
