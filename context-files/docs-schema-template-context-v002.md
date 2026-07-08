# Reusable Docs Repo Schema — Context File v002

Paste this into a new AI session to bootstrap a new project's private
docs repo, or to revisit/change the schema itself. This file is
de-branded — it describes the structure and reasoning, not any
specific project. It was developed across two projects: NoiseToSignal
(consumer web app) and LeadPilot (B2B AI agent), and is meant to be
reused for whatever comes next.

**v002 change from v001:** added the "Code-repo-side public files"
distinction below — a public-facing `ROADMAP.md` (and potentially
other build-in-public artifacts) belongs directly in the `<project>`
code repo, not just in the docs repo's `public/` folder. See that
section for the reasoning. Everything else carried over unchanged
from v001.

## The core idea

Every project gets two repos, both private:

1. **`<project>` (code)** — the actual application/agent code.
2. **`<project>-docs` (docs)** — planning, research, security,
   decisions, and AI-session context. Never merged into the code repo.

**Why two repos, not one:** code repos tend to get shared more widely
over time than planned — future contractors, CI integrations,
eventual open-source release, forks. Git history is permanent; once
something lands in a commit, it's recoverable from history even after
the file is deleted. Docs repos hold exactly the content you don't
want inheriting that wider audience: security thresholds, account-
linked URLs, compliance timelines, partnership/equity details, raw AI
transcripts. Keeping them in a separate repo means collaborators can
be granted access to one without automatically getting the other.

**Convenience without merging:** if a collaborator needs "one repo to
work with," add them as a collaborator to both repos rather than
merging — it's roughly the same friction after initial setup, without
giving up the security separation. A git submodule is a middle-ground
option (single top-level clone, still separate history/access) but
adds git friction (people forget to `submodule update`) — prefer plain
two-repo-with-collaborator-access unless there's a specific reason to
want a single `git clone`.

## The folder schema (19 folders)

Each folder gets a `README.md` with two required sections: **"What
belongs here"** and **"Suggested file naming."** Optionally, a
**"Current state"** or **"Notes"** section capturing what's actually
been decided so far, so the README isn't just a template — it's a
living reference.

| Folder | Purpose |
|---|---|
| `prd/` | Product/agent requirements per phase. One file per phase, `.md` or `.docx`. Never delete old versions. |
| `mvp/` | Internal scope checklist — in/out of scope, checked off only when built AND verified. |
| `architecture/` | System design — data flow, component/tool map, state schema. Written for someone picking the project up cold. |
| `tech-stack/` | Technologies used/considered, versions, why chosen, what was rejected and why. OK to leave as a stub with options listed if undecided — don't let stack indecision block other planning. |
| `commands/` | Every setup/dev/deploy command + required env vars (names only, never values). |
| `decisions/` | Numbered decision log: what, why, alternatives rejected, status (Active/Superseded). This is the single highest-value folder for "why does it work this way" and doubles as public/ source material. Keep the "how many decisions" count accurate in the README prose whenever you append — it's an easy thing to let go stale. |
| `research/` | Competitive analysis, API/platform research, market context. |
| `compliance/` | ToS, GDPR/CCPA, data handling agreements, any platform-specific verification process (e.g. OAuth verification). |
| `security/` | See expanded structure below — this folder deserves more than a single README once a project is agent-driven or handles sensitive data. |
| `testing/` | See expanded structure below. |
| `settings/` | Every configurable value: default, options, storage, and — importantly — which "settings" are actually security-relevant constants that should stay hard-coded rather than user-editable. |
| `governance/` | Ownership, roles, decision rights, partnership terms. See note below — highest-priority folder to fill in for real once there's more than one owner. |
| `roadmap/` | Internal draft of the public-facing phase roadmap, written in outcome language (no tool/API names). Feeds both `public/` (docs repo) and the code repo's `ROADMAP.md` (see below) — this folder is the draft, not the final published artifact. |
| `release-process/` | Deployment runbook, rollback procedure, pre-deploy checklist. |
| `monitoring-observability/` | Logging strategy, alerting, uptime/schedule monitoring. Explicitly separate success/outcome logs from error logs (a pattern worth reusing: `sync_log` vs `error_log`). |
| `public/` | Conventions + a sanitization checklist for turning private material into public "build in public" content (social posts, blog drafts). The private repo itself is never published; this folder is the curated staging layer — distinct from the code repo's `ROADMAP.md`, which is the finished, already-sanitized artifact (see below). |
| `ai-conversations/` | Saved AI chat transcripts, subfoldered by model (`claude/`, `gpt/`, etc.), with a required summary header (date, model, summary, key outputs, key decisions). Never paste credentials into a transcript that gets saved here. |
| `context-files/` | Versioned "paste this to restore context" files (`<project>_context-v001.md`, incrementing, never overwritten). Keep each under ~800 lines. This folder is also where this schema-template file itself lives. |
| `installation/` | Local dev environment setup guides, separate from `commands/` (which is day-to-day usage, not first-time setup). |

## Code-repo-side public files (new in v002)

Build-in-public content has two layers, not one:

1. **Staging/curation (docs repo, private):** `public/README.md` and
   its sanitization checklist — drafts of posts, a "safe to share"
   rewrite of the decisions log. Never published as-is; this is where
   raw material gets turned into something safe to say out loud.
2. **Published artifact (code repo, visible):** a `ROADMAP.md` at the
   root of `<project>` (the code repo) — the actual finished,
   already-sanitized roadmap, written in outcome language, with no
   tool/API names or internal thresholds. This is what a visiting
   collaborator, future contractor, or (if the repo's visibility ever
   changes) the public actually sees. Link to it from the code repo's
   `README.md` under a short "Roadmap" section.

Don't skip straight to publishing drafts from `public/` into
`ROADMAP.md` without running the sanitization checklist first — the
two-layer split exists specifically so unsanitized material never
lands somewhere more widely visible than intended.

## Expanded `security/` structure (worth doing from day one, not just once a project is defensive-security-mature)

  README.md                          Overview + file index
  threat-model.md                    Named failure modes, worst-case impact, safeguard, verification method
  incident-response-plan.md          Contain -> assess -> notify -> fix -> review steps
  dependency-vulnerability-policy.md Scanning cadence, CVE review process
  secrets-rotation-runbook.md        Per-credential rotation procedure
  pen-test-checklist.md              Adversarial/abuse-case checklist, companion to testing/'s functional suite

If the project is agent-based (LLM with tool access), the threat model
should treat prompt injection as a first-class threat category, not an
afterthought — model it the same way you'd model SQL injection: named
attack surface, specific safeguard, specific regression test.

## Expanded `testing/` structure

  README.md                 Overview + the evidence standard (below)
  eval-suite.md / test-plan Actual test cases with expected output written BEFORE the fix/feature is built
  ci-strategy.md            What runs automatically vs. stays manual, and why
  definition-of-done.md     Explicit checklist — a feature isn't done until this passes
  known-issues-log.md      Open bugs/gaps, distinct from decisions/ (which records resolved decisions)

**The evidence standard** (worth restating in every testing/README.md):
a fix or feature is only "done" when there's actual evidence — a log
line, literal output, a specific count — not "it looks right" or "the
code should work." Apply this to both human testing and any AI coding
assistant working on the codebase.

## `governance/` — do this even if it feels premature

The single most commonly-skipped folder, and the one most likely to
cause real damage if skipped: ownership/equity split, IP assignment,
decision-making authority, and what happens if a partner's involvement
changes. Easy to defer because writing it down feels awkward with
someone you trust — that's exactly why it should be a named folder
from the start, even as a stub that says "needs a real conversation
and, ideally, legal review." Never treat anything in this folder as
binding without actual legal review once real equity/revenue/IP is at
stake.

## Code-repo-side files (not in the docs repo — these live in `<project>` itself)

  README.md              Written for outside readers, not just you
  ROADMAP.md              Public-facing roadmap — see "Code-repo-side public files" above
  LICENSE                Even a "proprietary, all rights reserved" placeholder beats no file
  CONTRIBUTING.md         Dev workflow, review expectations
  CHANGELOG.md            Keep a Changelog format
  SECURITY.md             Public-safe responsible-disclosure policy — contact info only, no internal detail
  CODEOWNERS              Path-based review requirements, especially for security-sensitive paths
  .github/ISSUE_TEMPLATE/ Bug report + feature request templates
  .github/PULL_REQUEST_TEMPLATE.md
  .github/workflows/      CI, even as a stub noting "not yet configured" — the shape should exist before the content does
  .env.example            Key names only, never real values
  .gitignore              Cover .env (not just .env.local!), OS cruft, and stack-specific build artifacts once known

## Root README template (docs repo)

Every docs repo's root README should cover: what the project is (2-3
sentences), who owns it, why two repos exist (the security-separation
paragraph above, adapted), a folder-overview table, and an "Important"
section naming which folders contain genuinely sensitive material that
should never be published as-is.

## Open questions to resolve per-project (don't assume — ask)

- What does the project actually do? (Shapes compliance/, research/,
  and how heavy security/ needs to be from day one.)
- Where do the two repos live, and what's the exact naming (`<project>`
  vs `<project>-docs`, or some other convention already in use)?
- Solo owner or partnership? (Determines how much governance/ matters
  immediately vs. later.)
- Tech stack known or undecided? (OK to proceed with everything else
  stack-agnostic if undecided — don't block planning on this.)
- Should the `public/` folder (and the code repo's `ROADMAP.md`) be
  scaffolded immediately, or added later once there's something worth
  sharing?
- License approach — proprietary placeholder, open source, or
  genuinely undecided?
- Full professional code-repo scaffold (LICENSE/CONTRIBUTING/CI/etc.)
  or docs-only for now?
- If the code repo already exists with its own history and
  collaborators, check for pre-existing uncommitted work before
  staging anything — commit only the files you actually added, never
  sweep up someone else's in-progress changes into your commit.

## Notes for whoever picks this file up next

- This schema originated from NoiseToSignal (a consumer web app) and
  was expanded while building LeadPilot (a B2B AI agent) — the core
  19-folder structure held up across very different project shapes,
  which is a decent signal it generalizes. Adapt folder *contents* to
  the domain (e.g. `compliance/` looks very different for an
  agent handling financial documents vs. a YouTube client), but the
  folder *names and purposes* shouldn't need to change much.
- If you find yourself wanting a 20th folder, or another code-repo-side
  convention like `ROADMAP.md` was in v002, add it here too (bump to
  v003) so the next project benefits from the same gap analysis
  instead of repeating it from scratch.
