# Definition of done

A feature, tool, or fix is done only when all of the following are
true — matching the evidence-before-fix standard carried over from
NoiseToSignal.

## For a tool (`fetch_all_leads`, `get_contact_history`,
## `verify_drive_contents`, `dispatch_slack_handoff`,
## `search_communications`, `update_lead_sheet`,
## `initiate_backoffice_call`)

- [ ] Implemented against the actual API (not just a mock) at least
      once, with real (or realistic sandboxed) credentials
- [ ] Unit tests pass against mocked responses, including error cases
      (API timeout, empty result, malformed data)
- [ ] Output shape matches what the system prompt/architecture expects
      downstream
- [ ] Relevant eval-suite.md case(s) pass with the tool wired in
- [ ] If the tool has a real-world side effect (`dispatch_slack_handoff`,
      `initiate_backoffice_call`, `update_lead_sheet`, or any
      lead-facing outreach send), it rejects execution without a
      valid, single-use rep-approval token — verified with an actual
      rejected call, not just code inspection

## For a prompt or prioritization-logic change

- [ ] All six eval-suite.md cases still pass
- [ ] If the change could affect the security guard or the
      execution-gating rule, the relevant pen-test-checklist.md items
      pass too
- [ ] Change and reasoning logged in decisions/README.md (the
      decisions log)

## For a bug fix

- [ ] Root cause identified and stated explicitly (not "seems fixed")
- [ ] A new eval-suite.md or pen-test-checklist.md case added that
      reproduces the original bug and now passes
- [ ] Verified with actual output (log line, JSON payload, Slack
      message received) — not code inspection alone

## Not done

- "It should work" without having actually run it
- Passing eval-suite.md Cases 1-2 but not Case 3, 5, or 6 (security
  and execution-gating cases are not optional)
- Code reviewed but never executed against real or mocked API calls
