# TCPA / CAN-SPAM / Do-Not-Call research and gap analysis

Requested as one of Step 5's three open items (`mvp/README.md`). This
is legal-*adjacent* research, not legal advice — it summarizes public
regulatory requirements and checks them against the real code as of
`main` (commit `dcd9b6c`), the same "verify by testing/reading the real
thing, not assuming" discipline Decision 038's security review used.
**A qualified attorney must review this before LeadPilot contacts a
real lead** — see `compliance/README.md`'s existing note. Nothing here
should be read as "compliant" or "non-compliant" in a legal sense; it's
"here's what the rules say, here's what the code currently does, here's
the gap."

## Open question this whole analysis depends on

The PRD (`prd/LeadPilot_PRD_v1.06.md` line 142) says "LeadPilot is an
AI agent for **B2B Sales and Business Development teams**" — but that
describes who *buys/uses* LeadPilot (a B2B sales org), not necessarily
who the *leads being contacted* are. The actual eval-case content
("bank statements," "prequalification," Case 1's "financial records")
reads like individual loan/mortgage applicants — i.e., real consumers
reachable at personal cell numbers — not company switchboard numbers.

**This distinction changes which rules apply**, most importantly the
Do-Not-Call Registry's B2B exemption (see below). Nothing in `prd/`,
`architecture/state-schema.md`, or the `Lead` model
(`src/leadpilot/models/leads.py`) records whether a given lead is a
business entity or an individual consumer. **This needs a direct
answer from Marc/Abdoul before the DNC section below can be resolved
one way or the other** — flagging it here rather than guessing.

## 1. TCPA (Telephone Consumer Protection Act) — calls and texts

**What it requires (general, current as of 2026-07):**
- Autodialed or prerecorded/artificial-voice calls/texts to a cell
  phone need the consumer's prior consent first. A 2026-02 Fifth
  Circuit ruling (*Bradford v. Sovereign Pest Control*) held the TCPA
  itself only requires *prior express consent* (oral or written) for
  prerecorded telemarketing calls, not the FCC's stricter *written*
  standard — but that ruling only binds the Fifth Circuit; the FCC's
  written-consent rule otherwise remains in effect nationally. Treat
  written consent as the safe default until this is briefed by counsel
  for whichever jurisdictions LeadPilot's leads are actually in.
- Opt-out: as of the FCC's April 2025 rule change, a recipient can
  revoke consent through **any reasonable method** (replying STOP,
  emailing, calling, a website form) — not just a keyword reply — and
  the business must honor it within 10 business days, "as soon as
  practicable."
- Violations carry statutory damages of $500–$1,500 per message/call,
  and TCPA class actions are common.

**How this maps to LeadPilot's actual design:**
- `initiate_lead_call` (Decision 016) never dials anything — on
  approval it copies a phone number to the rep's clipboard and the rep
  manually places the call. A human manually dialing is a materially
  different TCPA posture than an autodialer/prerecorded-voice call —
  this is a real point in LeadPilot's favor, already true today, not a
  gap. (Still not a blanket exemption — a human making a high volume of
  calls off a queue can still raise other issues, e.g. DNC below.)
- `send_lead_text` (`src/leadpilot/tools/send_lead_text.py:123`) calls
  `twilio_client.messages.create()` directly — this is exactly the
  kind of automated, API-driven text send TCPA's consent rules are
  aimed at. **The rep clicking "approve" is not the same thing as the
  lead having given TCPA consent to be texted** — nothing in this tool,
  the `Lead` model, or `contact_history` records whether/when/how the
  lead consented to receive texts.
- **No opt-out handling exists anywhere in the codebase.** Confirmed by
  grep: no inbound Twilio webhook route, no `/sms` endpoint, no
  `opt_out`/`unsubscribe`/`STOP` handling in `src/leadpilot/*.py` or
  `src/leadpilot/tools/*.py`. If a lead replies "STOP" to a LeadPilot
  text today, Twilio's own carrier-level opt-out handling may catch it
  silently (Twilio auto-suppresses future sends to a number that
  replied STOP, if Advanced Opt-Out is enabled on the account — not
  confirmed either way), but **LeadPilot itself never learns this
  happened**. The agent can draft another text to that same lead on
  the next hourly run, and a rep — with no way to see the lead opted
  out — could approve it. That's a live compliance gap, not a
  theoretical one.

## 2. CAN-SPAM Act — email

**What it requires:**
- Accurate From/Reply-To/routing info (not deceptive).
- Non-deceptive subject line.
- Clear disclosure that the message is an advertisement, if its
  primary purpose is commercial.
- A valid physical postal address in the message.
- A clear, working opt-out mechanism, honored within 10 business days;
  can't sell/transfer the email address after someone opts out.
- Up to $53,088 per individual email in violation.
- Applies regardless of B2B vs. consumer — CAN-SPAM covers "commercial
  electronic mail," not just consumer marketing.

**How this maps to LeadPilot's actual design:**
- `send_lead_email` (`src/leadpilot/tools/send_lead_email.py`) sends
  from the approving rep's real Gmail account, so From/Reply-To
  accuracy is inherently satisfied — that part's fine.
- The message body (`json.dumps({"subject": subject, "body": body,
  "to": ...})`, built into a plain `MIMEText` at
  `execute_send_lead_email`, line 146) has **no physical postal
  address, no unsubscribe link/instructions, and no advertisement
  disclosure** — these are drafted freeform by the agent/rep with
  nothing in the tool enforcing their presence.
- **No suppression/opt-out list exists** — same gap as texts. If a lead
  replies asking to be removed, or clicks a link that doesn't exist
  (there is no unsubscribe link to click), nothing in LeadPilot
  prevents the next hourly run from drafting another email to them.

## 3. Do-Not-Call — National Registry and internal list

**What it requires:**
- **National DNC Registry**: telemarketing calls to a number on the
  Registry are prohibited unless an exemption applies (e.g. an
  existing business relationship). Callers must scrub calling lists
  against the Registry periodically and pay for registry access.
- **B2B exemption**: calls soliciting a *business* (not marketing
  nondurable office/cleaning supplies) are generally exempt from the
  Registry. This exemption does **not** extend to an employee's
  personal mobile number, a home-based business, a mixed-use line, or
  any campaign that blends consumer and business targets.
- **Internal/entity-specific DNC list**: independent of the National
  Registry, a business must maintain its own do-not-call list honoring
  any individual's direct request not to be called again — this
  applies whether or not the B2B exemption covers the call in the
  first place.

**How this maps to LeadPilot's actual design:**
- Whether the Registry applies at all hinges entirely on the open
  question above (are contacted leads businesses or individual
  consumers). Unresolved.
- **The internal/entity-specific DNC list requirement is not
  B2B-exempt and applies regardless** — and LeadPilot has no mechanism
  for it at all. There's no field on `Lead`
  (`src/leadpilot/models/leads.py`) or anywhere in `contact_history`
  for "this person asked not to be contacted again." A rep who
  verbally hears "please don't call me again" and logs that fact
  nowhere in LeadPilot has no way to stop the agent from drafting the
  next outreach to that same lead next hour.
- No National DNC Registry scrub step exists anywhere in
  `fetch_all_leads` or before `initiate_lead_call`/`send_lead_text`
  stage a draft.

## Gap summary (for Marc/Abdoul to prioritize, not yet built)

| # | Gap | Files | Severity |
|---|---|---|---|
| 1 | No B2B-vs-consumer classification on leads, so DNC Registry applicability is unknown | `models/leads.py` | Blocks resolving #3 below |
| 2 | No consent tracking (whether/when/how a lead consented to be called/texted) | `models/leads.py`, `contact_history` | High — TCPA needs documented consent |
| 3 | No opt-out/suppression list checked before drafting or sending calls/texts/emails | `send_lead_text.py`, `send_lead_email.py`, `initiate_lead_call.py` | High — this is the entity-specific DNC list requirement, not B2B-exempt |
| 4 | No inbound SMS webhook to capture STOP replies into LeadPilot's own state | none exists | High — currently relies entirely on Twilio's own (unconfirmed) carrier-level handling |
| 5 | `send_lead_email` body has no unsubscribe link, physical address, or ad disclosure | `send_lead_email.py` | Medium — CAN-SPAM requires all three |
| 6 | No National DNC Registry scrub step | `fetch_all_leads.py`, tool layer | Depends on #1 |

## Recommendations (proposed only — not implemented, no code changed)

**Blocked on a fact only Marc/Abdoul know, not an engineering choice:**

0. **Are the leads LeadPilot contacts individual consumers, or
   businesses?** Decides how urgent #4 below is and whether the DNC
   Registry's B2B exemption applies at all. Needed before that item can
   be scoped, let alone built.

**Buildable now, in priority order:**

1. **Suppression/opt-out table** (closes the consent-tracking and
   suppression-list gaps together — the highest-leverage single piece,
   since it's channel-agnostic and gates all three outreach tools). A
   small dedicated table, matching the existing
   `lead_action_locks`/`sheet_cell_locks` pattern rather than
   overloading an existing one: `lead_id`, `channel`
   (call/text/email/all), `reason`, `source`, `created_at`. Checked as
   a hard gate inside `send_lead_text`, `send_lead_email`,
   `initiate_lead_call` at **draft time** (`gate.create_draft`), not
   just execute time — a suppressed lead never even reaches the rep's
   approval queue, rather than relying on the rep to notice and reject
   it. New mechanism, so it gets a Decision log entry alongside the
   migration, not just silent code.
2. **Inbound Twilio webhook** — a new route (e.g.
   `/webhooks/twilio/inbound-sms`) Twilio calls when a lead replies;
   detects STOP-style replies and writes into the suppression table
   from #1, rather than trusting Twilio's own (unverified)
   account-level handling silently. Needs a Twilio Console
   messaging-webhook URL pointed at a real deployed instance, so it's
   only fully testable once there's a live Render deploy, not just
   locally.
3. **CAN-SPAM footer on emails** — append a fixed footer in
   `execute_send_lead_email` with unsubscribe instructions and a
   physical postal address, rather than leaving the agent's drafted
   `body` to remember both every time. The address itself has to be
   the sales org's real one — a fact from Marc, not something to
   invent.
4. **Consent provenance fields** — add `consent_source`/
   `consent_captured_at` to `Lead`, populated from whatever the sales
   org's own upstream process already captures. Lower priority, more
   of a documentation/audit-trail policy question (does LeadPilot need
   to *verify* consent exists, or is it reasonable to trust the sales
   org already has it and record where it came from?) than an
   engineering one.
5. **National DNC Registry scrub** — blocked on item 0. If leads turn
   out to be consumers, this needs a paid registry-check integration
   (a new external account, similar to the Step 0 accounts work)
   before a lead is treated as callable/textable.

## Notes

- This document doesn't cover state-level "mini-TCPA" laws (e.g.
  Florida, Oklahoma, Washington impose stricter consent/consent-scope
  rules than the federal TCPA) — flagged here as existing, not
  researched in depth; relevant once it's known which states' leads
  LeadPilot will actually contact.
- Not a substitute for attorney review, per `compliance/README.md`'s
  standing note — this is the technical/factual groundwork for that
  review, not the review itself.
