---
name: customer-support-release-notes-feedback-updater
description: Update StaffAny Customer Support release notes using validator feedback, confidence score, evidence gaps, and user feedback. Use after customer-support-release-notes-validator returns revise, or when a teammate asks LaunchBot to update CS release notes from validation feedback.
---

# Customer Support Release Notes Feedback Updater

Use this skill to revise Customer Support release notes after validation or teammate feedback.

## Inputs

```text
original_release_notes: <five-section release note>
validation: <customer-support-release-notes-validator output>
source_evidence: <Jira/Slack/Pantheon/help article summaries>
user_feedback: <optional Slack/thread feedback>
```

## Update Rules

- Preserve the exact five-section format:
  - Module
  - What's new
  - How this helps users
  - What's needed to be setup
  - Help article link
- Apply validator `Required Changes` in priority order.
- Update only with evidence-backed facts. Do not invent UI labels, setup steps, availability, module names, or help article links.
- Keep `What's new` focused on the UI/UX delta from previous behavior to new behavior.
- Keep `How this helps users` focused only on customer, admin, manager, or employee value. Remove CS, support-agent, triage, or internal-team explanations from that section.
- Keep each section concise enough for CS to scan quickly.
- If requested feedback is unsupported by the evidence, put it under `Remaining Needs Check` instead of adding it to the release note body.
- If the validator decision was `blocked`, do not produce a revised release note unless the missing blocker evidence is supplied.

## Output Contract

```text
Updated Release Notes:
- Module
  ...

- What's new
  ...

- How this helps users
  ...

- What's needed to be setup
  ...

- Help article link
  ...

Changes Applied:
- <validator or user feedback addressed>

Remaining Needs Check:
- <none or exact evidence gap>

Validator Handoff:
Re-run customer-support-release-notes-validator before marking ready.
```

## Guardrails

- Do not expand the note into a changelog, help article, or marketing announcement.
- Do not expose raw Jira descriptions, private URLs, customer names, PII, internal app names, or implementation-only details.
- Do not lower the standard because the previous score was close to passing. The revised draft still needs a fresh validator pass.
