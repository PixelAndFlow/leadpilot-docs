# AI Conversation — Step 0 walkthrough, abdouls-branch merge documentation, and the Sheets/Drive access-model pivot (Decisions 026-028)

Date: 2026-07-11
Model: Claude (Sonnet 5, Cowork mode)
Summary: Marc connected both the `leadpilot` and `leadpilot-docs`
repos and asked Claude to walk him through Step 0 (accounts and
access) step by step. That started as a straightforward Google Cloud
setup — project, service account, a shared test sheet — but surfaced
a real gap along the way: `mvp/README.md`'s Step 0 checklist assumed
an OAuth client for Sheets even though Abdoul had already built the
Sheets connector against a service account (Decision 024, flagged but
never confirmed by Marc). Claude walked through that tradeoff, Marc
confirmed the service-account plan, and Claude propagated the
confirmation across five docs. Marc then separately asked Claude to
document that he'd just merged `abdouls-branch` into `main`. Finally,
while creating test seed data and setting up a live Google Sheet for
Step 0, Marc raised a bigger question: can the build plan handle an
agent that needs to work across several sheets and jump to a new one
"without notice"? That question, worked through carefully, ended in
Marc reversing Decision 024 entirely — Sheets and Drive access moves
to per-rep OAuth (`drive.file` scope + Google Picker) instead of a
shared service account, the hourly batch run becomes per-rep, and a
new tool (`fetch_ad_hoc_sheet`) is added for on-demand sheet lookups.
Marc asked for all of this to be documented across the PRD, MVP
checklist, decisions log, and this conversation log itself.

Key outputs:
- `prd/LeadPilot_PRD_v1.05.md` — new PRD version: `fetch_all_leads`,
  `verify_drive_contents`, and `update_lead_sheet` now authenticate
  per rep via OAuth instead of a service account; adds
  `fetch_ad_hoc_sheet`; rewrites section 3e (connector architecture)
  as rep-scoped; adds Eval Case 11 (per-rep access boundary); adds a
  `DATA ACCESS GUARD` to the system prompt
- `decisions/README.md` — Decision 024 confirmed, then marked
  Superseded the same day; Decisions 026 (access model), 027 (per-rep
  batch run, updates 022/025), and 028 (new tool) added
- `mvp/README.md` — Step 0 item 1 rewritten twice (once for Decision
  024, once reversed for Decision 026); tools list updated to eleven;
  Step 1's `GoogleSheetsConnector` entry flagged as needing rework;
  Step 2 given four new concrete items; Step 0's "abdouls-branch merged
  to main" status corrected
- `architecture/README.md` and `architecture/state-schema.md` —
  tool-map diagram updated for per-rep OAuth and the per-rep batch
  run; sketched a new `rep_google_credentials` table; flagged
  `agent_run_locks` needing to move from a singleton to a per-rep
  mutex
- `tech-stack/stack-overview.md` — Google integration bullet rewritten
  a third time (service-account-only → service-account-for-Sheets/
  OAuth-for-Gmail → per-rep OAuth for all three); Scheduler section
  updated for the per-rep loop; Google Picker API added to the stack
- `testing/eval-suite.md` — Case 11 added
- `commands/README.md`, `security/secrets-rotation-runbook.md`,
  `LeadPilot/.env.example` — env var docs updated across all three
  passes (service account → service account + Gmail OAuth → unified
  per-rep OAuth), each superseded section left as a visible trail
  rather than silently overwritten
- `LeadPilot/README.md`, `LeadPilot/CHANGELOG.md` — status sections
  corrected to reflect the `abdouls-branch` merge (commit `cc4c8ac`)
- `LeadPilot/seed-data/leadpilot_test_leads_sheet_a.csv` — 15 rows of
  fictional lead data for Marc to load into a test Google Sheet

Key decisions (see decisions/README.md for the canonical log):
- Decision 024 confirmed, then reversed the same day: a shared
  service account is the *wrong* fit once the requirement is "a rep's
  own Google permissions decide what LeadPilot can touch" — see
  Decision 026 below.
- Decision 026 — Sheets and Drive access moves to per-rep OAuth using
  the `drive.file` scope plus the Google Picker API, not a service
  account and not the broader `spreadsheets`/`drive` scopes. Each rep
  does a one-time consent; LeadPilot stores their refresh token and
  uses it for both the hourly batch run and on-demand lookups.
  `drive.file` was chosen specifically because it only grants access
  to files the rep explicitly picks — matching "nothing pre-shared,
  the rep decides" far more precisely than a blanket scope, and
  avoiding Google's sensitive-scope app-verification review.
- Decision 027 — the hourly batch run becomes per-rep instead of one
  global run, since there's no longer one shared "designated
  spreadsheets" list once access is scoped to what each rep
  personally connected. `agent_run_locks`' singleton mutex needs to
  become per-rep to match (flagged, not yet built).
- Decision 028 — a new tool, provisionally `fetch_ad_hoc_sheet`, lets
  a rep hand LeadPilot a sheet mid-session for an immediate one-off
  read, closing the gap that none of the existing ten tools covered
  "check this sheet right now."
- The existing Step 1 `GoogleSheetsConnector` (service-account-based)
  is not thrown away — it's flagged as needing rework in Step 2, and
  Marc's already-created Google Cloud project/service account stay in
  place to back local dev until that rework lands.

--- transcript below ---

**Marc:** Said he's building an agent with a collaborator in class and
has two repos to work with: `leadpilot` and `leadpilot-docs`.

**Claude:** Asked whether the repos are local, on GitHub, or both.
Marc said both. Claude requested folder access for `LeadPilot`
(already connected), then requested `leadpilot-docs` as well since
only the code repo was mounted. Read both repos' READMEs, the PRD
build order, and the git log to orient: `leadpilot` (code, pre-build
per its own README) and `leadpilot-docs` (private planning repo — PRD
up to v1.04, decisions log, architecture, security, testing).

**Marc:** Said he was about to do "Step 0" from the PRD readme and
wanted to go through it step by step.

**Claude:** Found Step 0 actually lives in `mvp/README.md`'s build
order, not `prd/README.md` — flagged the mismatch. Read `mvp/README.md`
and found Step 0's Google Cloud item ("OAuth client, Sheets/Drive/Gmail
scopes") was stale: Abdoul had already built `GoogleSheetsConnector`
against a service account (Decision 024), and that decision was
explicitly flagged in the docs as "not yet confirmed by Marc." Before
starting the walkthrough, asked whether to confirm Decision 024,
discuss it, or skip and move on.

**Marc:** Wanted to discuss it first.

**Claude:** Laid out the tradeoff: a service account is simpler for an
unattended hourly Cron Job with no per-rep identity to bind to; OAuth's
main benefit (per-rep attribution in Sheets' own revision history) is
thin since `update_lead_sheet` writes are already attributed to the
approving rep at the LeadPilot layer (Decision 013). Recommended
confirming the service-account plan as-is.

**Marc:** Confirmed it, and asked Claude to write the reasoning into
the docs and update anything that referenced the old assumption.

**Claude:** Updated Decision 024's entry with the confirmation and
reasoning, removed it from the open-decisions list, and corrected five
other places that assumed OAuth covered Sheets: `mvp/README.md`'s Step
0 item, `commands/README.md`'s env var table, `tech-stack/stack-overview.md`'s
Google integration bullet, and `security/secrets-rotation-runbook.md`'s
secrets list.

**Marc:** Walked through the actual setup step by step with Claude:
created a Google Cloud project, enabled the Sheets API, created a
service account and downloaded its JSON key, and got the service
account's `client_email`.

**Claude:** At each step, gave the next concrete action and waited for
confirmation before moving on, per Marc's "step by step" request.

**Marc:** Asked Claude to create fictional seed data (people, companies,
info) for him to load into a test sheet, rather than making his own.

**Claude:** Created `seed-data/leadpilot_test_leads_sheet_a.csv` — 15
fictional leads matching `GoogleSheetsConnector`'s expected header row
(Name/Phone/Email/Company/Source/Status), with a couple of intentional
blank fields to exercise the connector's null-handling. Marc imported
it, shared the sheet with the service account, and got the Sheet ID.

**Claude:** Before wiring up the env vars, flagged that
`tests/test_google_sheets_connector_live.py` is hardcoded against a
specific 5-row fixture (a "John Doe" row, exactly one shared source
tag pair) — Marc's 15-row sheet wouldn't satisfy those exact
assertions. Offered three paths: build a matching fixture sheet, ask
Abdoul to share his original, or leave the test skipped for now.

**Marc:** Asked whether the hardcoded `"test_sheet"` key meant the
system was locked to one physical sheet.

**Claude:** Clarified: the string is just a dictionary key in
`GOOGLE_SHEETS_SOURCES`, not a pointer to one sheet — but the test's
assertions on row content (and its exact-equality check on the full
source list) do mean satisfying that specific test requires matching
data under that key. Not architecturally locked; just a narrow test.

**Marc:** Before deciding, raised the real question: "an agent
usually works on 1-4 sheets at a time. They may need to quickly jump
to another sheet without notice. Can the current build plan
accommodate this?"

**Claude:** Checked the actual code and docs rather than guessing:
`GOOGLE_SHEETS_SOURCES` supports any number of sources architecturally,
but (1) adding a sheet means editing a JSON env var and redeploying —
no runtime registration — and (2) column mapping is a single global
fixed schema across all sheets, despite the PRD's own problem
statement saying different marketing partners use different columns.
Asked Marc to clarify which gap he meant: faster onboarding of known
sheets, true on-demand/ad hoc access to sheets the agent wasn't
pre-configured for, or both.

**Marc:** True on-demand sheet selection.

**Claude:** Explained why that changes the access model, not just the
config: a service account can only read what's pre-shared with it,
which defeats "without notice" — the right mechanism is the rep's own
OAuth grant, the same reasoning Decision 024 already used to carve
Gmail out from the service account. Named a security upside (bounding
access to what that specific rep can already see is tighter than the
service account's blanket access) and two consequences: the OAuth
client Marc was about to create should also request Sheets scope, and
this is a new tool not in the PRD's existing ten, needing a PRD
revision, a decision entry, and an eval case. Asked whether to draft
that now or flag it and keep moving through Step 0.

**Marc:** Wanted to draft it now, and confirmed: rep's Google
permissions should decide access; sheets should never be pre-shared
except through the agent; and a shared service account was
specifically unwanted because Marc wants activity trackable per rep —
a real business reason for exactly the "thin benefit" Claude had
earlier weighed against OAuth for Decision 024.

**Claude:** Recognized this reverses Decision 024's conclusion, not
just extends it, and that it raises real architecture questions before
writing anything: how a rep grants access in the first place, whether
the hourly batch run becomes per-rep, whether Drive follows the same
model, and how precisely the OAuth scope should be bounded. Asked all
four before proceeding.

**Marc:** One-time consent per rep (a stored refresh token, not
re-auth every time); the batch run becomes per-rep; yes, Drive follows
the same model; wanted to discuss the scope-precision question further.

**Claude:** Laid out `drive.file` + Google Picker (rep explicitly
selects specific files) versus the broader `spreadsheets`/`drive`
scopes (standing access to everything the rep can see), recommending
`drive.file` — it matches "deliberate, trackable, never pre-shared"
much more precisely, and avoids Google's sensitive-scope
app-verification review.

**Marc:** Agreed, and asked Claude to continue: document the
requirements in the PRD, MVP checklist, and decisions log; add this
conversation to `ai-conversations`; and update every other document
that needed the new configuration reflected. Asked if Claude had
questions before starting — Claude had none at that point and
proceeded.

**Claude:** In parallel with all of this, Marc had separately mentioned
merging `abdouls-branch` into `main` right before the session started.
Claude confirmed the merge in git history (commit `cc4c8ac`, authored
by Marc) and updated `mvp/README.md`'s Step 1 status, the decisions-log
attribution note, `LeadPilot/README.md`'s status section (which had
still claimed no application code existed), and `CHANGELOG.md`.

**Claude:** Wrote Decisions 026-028, PRD v1.05 (tool table, system
prompt, blast radius, Eval Case 11, and a rewritten section 3e),
updated `mvp/README.md`'s Step 0/1/2, `architecture/README.md` and
`state-schema.md` (tool-map diagram, a sketched `rep_google_credentials`
table, the per-rep mutex flag), `tech-stack/stack-overview.md` (Google
integration bullet rewritten a third time, Scheduler section, Picker
API added), `testing/eval-suite.md` (Case 11), `commands/README.md`
and `security/secrets-rotation-runbook.md` (env vars and secrets list
unified around the new OAuth client), and `LeadPilot/.env.example`
(annotated rather than deleting the still-functional Decision-024
variables, since Step 1's shipped code still depends on them until
Step 2's rework lands). Wrote this conversation log last.

(This is a condensed reconstruction of the conversation's substance,
not a raw system export — tool-call output and intermediate file reads
are summarized rather than pasted verbatim.)
