# testing/

Test plans, the eval card regression suite, CI strategy, and the
definition of done for LeadPilot. Builds on the evidence-before-fix
standard established on NoiseToSignal: a fix or feature is only done
when you can point to actual output proving it, not a description of
expected behavior.

## Files in this folder

  eval-suite.md            The PRD's 3 eval card cases as a standing
                            regression suite, plus cases added later
  ci-strategy.md           What runs automatically vs. what stays manual
  definition-of-done.md    When a feature/tool is actually finished
  known-issues-log.md      Open bugs/gaps, distinct from decisions/

## Relationship to security/pen-test-checklist.md

eval-suite.md and pen-test-checklist.md are companions: eval-suite.md
is the standing functional + security regression run before any
change ships; pen-test-checklist.md is the broader adversarial sweep
run before a launch or major release. Don't merge them — the eval
suite needs to be fast enough to run constantly, the pen-test
checklist doesn't.

## The evidence standard (carried over from NoiseToSignal)

A test only passes if you can point to actual evidence — a log line,
the literal JSON output, a specific count — not "it looked right" or
"the code looks correct." This applies to both human testing and any
AI coding assistant working on this codebase.
