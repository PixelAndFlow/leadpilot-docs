# context-files/

Context files used to restore full project state in any AI session.
Paste the current version at the start of any new conversation.

## Current file

  leadpilot_context-v003.md     Active — use this one
  leadpilot_context-v002.md     Previous — kept for reference
  leadpilot_context-v001.md     Previous — kept for reference

## Versioning convention

Versions increment by 0001 per update: v0001 → v0002 → v0003...

When updating: save a new file with the incremented version number.
Never overwrite the current version — always create a new file.

## How to use

1. Open a new AI chat (Claude, GPT, Gemini, etc.)
2. Paste the entire contents of leadpilot_context-v003.md
3. The AI has full context on what LeadPilot is, its tools, the
   system prompt, blast radius/security posture, MVP scope, and open
   decisions

## When to create a new version

- After a major planning session that changes decisions
- After completing a build phase
- After receiving stress-test feedback from another LLM
- Before starting a new build phase
- When the active version gets meaningfully out of date

## Other reusable context files in this folder

  docs-schema-template-context-v002.md     Active — use this one
  docs-schema-template-context-v001.md     Previous — kept for reference

      A de-branded, reusable version of the entire docs schema (this
      repo's structure) — not LeadPilot-specific. Paste this into a
      future session to bootstrap a new project's docs repo, or to
      revisit/change the schema itself. v002 added the code-repo-side
      public ROADMAP.md convention.

  leadpilot_interface_design_context-v001.md     Active — use this one

      Scoped tightly to Step 3 (dashboard) design work, not general
      project restore — screen list, real tool output data shapes, the
      approval-gate interaction pattern that constrains every screen,
      and the locked Jinja2/htmx (not React SPA) tech constraint. Paste
      this into a session with a design-focused LLM instead of the
      general leadpilot_context file above.

## Size target

Keep context files under ~800 lines so they fit comfortably in an AI
context window without crowding out the actual conversation.
