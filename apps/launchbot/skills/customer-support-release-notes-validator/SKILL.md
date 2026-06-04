---
name: customer-support-release-notes-validator
description: Validate StaffAny Customer Support release notes with confidence scoring and evidence-based reasoning. Use immediately after customer-support-release-notes-generator drafts release notes, or when reviewing CS release notes for shipped Jira KER tickets.
---

# Customer Support Release Notes Validator

Use this skill as the quality checkpoint after a Customer Support release note draft is generated.

## Inputs

```text
jira_key: <KER-key>
jira_status: <status>
launch_priority: <P1 | P2 | P3 | P4 | blank>
source_evidence: <Jira/Slack/Pantheon/help article summaries>
draft: <release notes draft>
help_article_link: <URL | draft URL | TBD>
```

## Validation Rules

- The draft must keep exactly these sections: Module, What's new, How this helps users, What's needed to be setup, Help article link.
- `What's new` must describe user-visible UI/UX delta from old behavior to new behavior.
- `How this helps users` must explain customer, admin, manager, or employee value only. It must not explain how the change helps CS, support agents, triage, or internal teams.
- Evidence must support every product behavior, UI label, setup step, availability statement, and help article link.
- CS usefulness matters more than marketing polish: a support teammate should understand what changed, what users will see, and what setup to check.
- Use `needs-check` for incomplete but non-blocking evidence. Use `blocked` for unsafe, unverified, or non-shipped claims.

## Scoring

Return a score from `0` to `100`.

- Evidence grounding: `0-30`
- UI/UX delta clarity: `0-25`
- CS usefulness: `0-20`
- Setup and help article completeness: `0-15`
- Brevity, safety, and polish: `0-10`

Decision thresholds:

- `pass`: `85-100` and no blockers.
- `revise`: `70-84`, or fixable issues with enough evidence to correct the draft.
- `blocked`: below `70`, missing required evidence, unsafe claims, or a mandatory blocker.

Confidence bands:

- `High`: `85-100`
- `Medium`: `70-84`
- `Low`: `50-69`
- `Very low`: below `50`

## Mandatory Blockers

Set decision to `blocked` when the draft:

- Lacks shipped/done status or a clearly labeled preview request.
- Claims product behavior that is not backed by Jira, Pantheon, approved article evidence, or trusted screenshots.
- Omits any required release-note section.
- Invents a help article link, setup requirement, UI label, module, availability, or customer impact.
- Exposes raw Jira text, private URLs, customer names, PII, internal app names, or implementation-only details.
- Has no customer-visible behavior for CS to support.

Set decision to `revise` when `How this helps users` contains CS/support/internal-team value even if the rest of the draft is accurate.

## Output Contract

```text
Release Notes Validation:
Decision: <pass | revise | blocked>
Confidence Score: <0-100>
Confidence: <High | Medium | Low | Very low>

Evidence-Based Reasoning:
- <claim checked> -> <supporting evidence or gap>

Category Scores:
- Evidence grounding: <0-30> - <reason>
- UI/UX delta clarity: <0-25> - <reason>
- CS usefulness: <0-20> - <reason>
- Setup and help article completeness: <0-15> - <reason>
- Brevity, safety, and polish: <0-10> - <reason>

Required Changes:
- <change or none>

Ready for CS: <yes | no>
Next Skill: <none | customer-support-release-notes-feedback-updater>
```

## Handoff

- If decision is `pass`, the release note can move to CS/Product Lead review.
- If decision is `revise`, run `customer-support-release-notes-feedback-updater` with this validation output.
- If decision is `blocked`, do not revise by guessing. Gather the missing evidence first.
