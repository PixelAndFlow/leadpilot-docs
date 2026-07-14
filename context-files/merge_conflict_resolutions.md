# Merge conflict resolutions: marc-step2-split <- origin/abdouls-branch

Verified by actually running `git merge origin/abdouls-branch` on a throwaway
branch and inspecting every conflict. 4 files conflict; everything else
(including `locks.py` itself) merges automatically. Every conflict below is
"keep both sides" — nobody's change needs to be dropped.

After running `git merge origin/abdouls-branch`, open each file, find the
`<<<<<<< HEAD ... =======  ... >>>>>>> origin/abdouls-branch` block, and
replace the whole block (including the marker lines) with the text shown
below. Then `git add <file>` for each, and `git commit` to finish the merge.

---

## 1. `src/leadpilot/models/run_lock.py`

One conflict, in the module docstring only (the actual class bodies below
already merged cleanly — `AgentRunLock` keeps Abdoul's per-rep `rep_id`
primary key, `SheetCellLock` is untouched). Replace the conflicted docstring
block with:

```
"""Three distinct locks, serving three distinct failure modes from
security/threat-model.md:

1. LeadActionLock — per-lead duplicate-contact prevention. The named
   threat: "Two run cycles triggered in rapid succession against the
   same lead... a hot lead is dialed or texted twice." One row per
   lead; `last_action_committed_at` is checked and atomically updated
   *before* a new outreach draft is created for that lead (see
   leadpilot.locks.try_acquire_lead_action_lock) — Step 2 business
   logic decides the actual cooldown window, this table just holds the
   timestamp.

2. AgentRunLock — a per-rep mutex (reworked from a singleton, Decision
   027/032) so the same rep's hourly batch run can't overlap with
   itself (e.g. a slow run still executing when the next scheduled
   trigger fires) — while rep A's run and rep B's run proceed fully
   independently, since the batch job now iterates once per connected
   rep rather than running once globally (Decision 027). Without this,
   two overlapping runs for the same rep could both draft outreach for
   the same lead before either one's lead-action lock check would
   catch the other.

3. SheetCellLock — added Decision 034, closing a real race condition
   found in `GoogleSheetsConnector.commit_field_write`: two reps
   (or the same rep's overlapping runs) approving edits to the same
   spreadsheet cell around the same time could silently clobber each
   other, since a plain Sheets API `values().update()` has no
   built-in compare-and-swap. One row per in-flight write, keyed by a
   `"{source_id}:{row_ref}:{field_name}"` string rather than a FK,
   since the target isn't a LeadPilot-owned row — see
   leadpilot.locks.try_acquire_sheet_cell_lock /
   release_sheet_cell_lock, and connectors/base.py's
   commit_field_write docstring for how this pairs with the
   *separate* optimistic expected-value check (this lock only
   protects against concurrent LeadPilot-originated commits; it can't
   catch someone editing the sheet directly in Google's own UI, which
   is what the expected-value check is for).
"""
```

---

## 2. `tests/test_locks.py`

One conflict, just the import block (everything below — Abdoul's rep-based
`test_run_lock_*` tests AND my `test_sheet_cell_lock_*` tests — already
coexist correctly with no other changes needed). Replace the conflicted
import block with:

```python
from leadpilot import auth, locks
from leadpilot.db import SessionLocal
from leadpilot.models.leads import Lead
from leadpilot.models.rep import Rep, RepSession
from leadpilot.models.run_lock import AgentRunLock, LeadActionLock, SheetCellLock
```

(Also update the module docstring at the very top from "for both locks" to
"for all three locks" if it wasn't already — trivial, not a real conflict.)

---

## 3. `src/leadpilot/connectors/google_sheets.py`

Two conflicts.

**Conflict A — imports.** Replace with:

```python
from leadpilot import google_credentials, google_oauth, locks
from leadpilot.connectors.base import (
    ChangesSummary,
    ConcurrentWriteError,
    FieldDiff,
    LeadRecord,
    LeadSourceConnector,
    StaleWriteError,
)
from leadpilot.connectors.google_drive import GoogleDriveClient, SPREADSHEET_MIME_TYPE
```

**Conflict B — `GoogleSheetsConnector.__init__`.** Both sides added an
independent, optional constructor parameter — combine them:

```python
    def __init__(
        self,
        session: Session,
        rep_id: uuid.UUID,
        sheets_service=None,
        drive_client: GoogleDriveClient | None = None,
    ):
        """`sheets_service` is normally left None (a real Sheets API
        client is built lazily from the rep's own OAuth token). Tests
        pass a fake here — same injectable-client pattern already used
        for Slack/Gmail/Twilio in the Step 2 tools — since this
        connector previously had no way to be exercised without live
        Google credentials and network access. `drive_client` is the
        same idea for the mimeType-filtering calls in list_sources()/
        _sheet_id_for() (Decision 033) — see tests/fakes.py.
        """
        self._session = session
        self._rep_id = rep_id
        self._service = sheets_service
        self._drive_client = drive_client
```

Everything below this (the mimeType-filtered `list_sources`/`_sheet_id_for`,
and the full `commit_field_write` with the lock + `StaleWriteError` check)
already merges automatically — no further edits needed in this file.

---

## 4. `src/leadpilot/google_oauth.py`

Three conflicts, all documentation/scope-list — no logic conflicts.

**Conflict A — module docstring, scope history.** Replace with:

```
Scope was drive.file only through Decision 026 — LeadPilot only ever
saw files a rep explicitly selected via the Google Picker, never their
whole Drive. Decision 033 added drive.readonly on top of that: the
drive.file per-item grant turned out not to extend to a folder's
contents (confirmed against the real API — granting a folder via
Picker does not grant visibility into files added to, or already
sitting in, that folder), which made verify_drive_contents unable to
do its actual job. drive.file is kept for the write path
(update_lead_sheet's commit_field_write) and for the deliberate
per-item consent UX on fetch_all_leads/fetch_ad_hoc_sheet;
drive.readonly is what verify_drive_contents actually reads through.
See Decision 033 for the full tradeoff and the note to revisit this
for a narrower alternative later.

Extended again 2026-07-13 by Marc (send_lead_email, Decision 032/030's
"Gmail-scope pairing" note) to add the two Gmail scopes both of Marc's
remaining Group B tools need — gmail.send for send_lead_email,
gmail.readonly for search_communications. Least-privilege choices:
gmail.send only allows sending, not reading/deleting a rep's inbox;
gmail.readonly allows reading message content/attachments for
search_communications but not sending or modifying anything — neither
is the broad gmail.modify or mail.google.com scope.

*** Any rep who already connected their Google account under an older
scope list needs to reconnect (re-run the "Connect Google Account"
flow) to grant drive.readonly and/or the Gmail scopes — Google won't
retroactively add them to an existing consent. Not yet reflected in
any UI messaging (Step 3 work); flag to affected reps manually until
then. ***
```

**Conflict B — refresh_token note, just below.** Replace with:

```
refresh_token in it, silently breaking storage. Widening SCOPES here
(either for drive.readonly or the Gmail scopes) means every rep who
connected before that widening is holding a refresh token that does
NOT cover the new scope — they must reconnect (redo the Connect
Google Account flow) before the tool that needs it will work for
them; there's no way to silently upgrade an already-issued token's
scope.
```

**Conflict C — the `SCOPES` list itself.** Replace with:

```python
SCOPES = [
    "https://www.googleapis.com/auth/drive.file",
    "https://www.googleapis.com/auth/drive.readonly",
    "https://www.googleapis.com/auth/gmail.send",
    "https://www.googleapis.com/auth/gmail.readonly",
]
```

---

## After resolving all 4

```
git add src/leadpilot/models/run_lock.py tests/test_locks.py \
        src/leadpilot/connectors/google_sheets.py src/leadpilot/google_oauth.py
git commit
```

Then run the real test suite before trusting any of this — none of it has
been executed against a real database yet (see `testing/known-issues-log.md`
Issue 006 in the docs repo for the exact commands and full context).
