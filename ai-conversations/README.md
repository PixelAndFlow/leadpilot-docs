# ai-conversations/

Saved logs of AI conversations used during the planning, design, and
building of LeadPilot. Organized by model so findings from different
tools can be compared.

## Folder structure

  claude/       Conversations with Claude (Anthropic)
  (add folders for other models as needed, e.g. gpt/, gemini/)

## What belongs here

- Exported or copy-pasted transcripts of significant conversations
- A summary header at the top of each file describing what was
  covered and what decisions or outputs came out of the session
- Stress-test results from running the context file through other LLMs

## Why save these

- Decisions made in AI conversations are easy to forget
- When starting a new session, summaries here restore context fast
- Comparing findings across different models surfaces blind spots
- Creates an audit trail of where product decisions came from

## File naming convention

  YYYY-MM-DD-topic.md

  Example:
  2026-07-07-docs-schema-scaffold-and-governance-planning.md

## File header format

  # AI Conversation — [topic]
  Date: [date]
  Model: Claude / GPT-4 / Gemini / etc.
  Summary: [2-3 sentences on what was covered]
  Key outputs: [files created, decisions made]
  Key decisions: [list of decisions that came from this session]

  --- transcript below ---

## Notes

- Save conversations that produced something important or that you
  might need to reference later.
- Never paste API keys, passwords, or credentials into an AI chat.
  Check transcripts before saving or sharing — LeadPilot's data (lead
  PII, financial documents) raises the stakes on this compared to
  NoiseToSignal.
