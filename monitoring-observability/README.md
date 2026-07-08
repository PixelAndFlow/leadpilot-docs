# monitoring-observability/

How you find out something broke, before a rep or a back-office
stakeholder does. Status: stub — fill in once tech stack and hosting
are chosen.

## What belongs here

- Logging strategy — what's logged per run (mirrors NoiseToSignal's
  sync_log/error_log separation: successes and outcomes in one place,
  system errors in another)
- Alerting — what triggers a notification to Marc/Abdoul directly
  (as opposed to a rep-facing notification)
- Uptime/schedule monitoring — how you know the hourly run actually
  ran, not just that no error was thrown
- Dashboards, if any, beyond the rep-facing queue view

## Suggested file naming

  logging-strategy.md      What's logged, where, retention
  alerting.md               What triggers an alert and to whom
  schedule-monitoring.md    Confirming the hourly run actually executed

## Minimum viable monitoring for Phase 1 (draft)

- [ ] Every run logs: start time, leads processed, Slack handoffs
      dispatched, any validation-layer interceptions (prompt injection
      attempts stripped)
- [ ] A missed scheduled run alerts Marc/Abdoul directly (not just
      logged silently)
- [ ] A validation-layer interception (i.e. a caught prompt injection
      attempt) alerts immediately — this is a security event, not
      routine noise
- [ ] Error log is separate from the success/outcome log, matching the
      sync_log/error_log pattern from NoiseToSignal

## Notes

- The validation-layer interception alert is the highest-priority item
  here — silently swallowing a caught injection attempt means you'd
  never learn the system is actually being probed.
