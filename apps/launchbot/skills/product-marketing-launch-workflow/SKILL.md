---
name: product-marketing-launch-workflow
description: Orchestrates StaffAny Launchbot product marketing launches from Jira KER release rows, sprint Roadmap values, launch-priority classification, help article work, Customer Support release notes, release-note validation, and launch status tracking. Use when planning or running LaunchBot PMM workflow steps for shipped KER tickets in Slack.
---

# Product Marketing Launch Workflow

Use this skill to turn Jira KER release rows into a launch tracker and material work queue.

## Required Inputs

- Jira KER rows with:
  - `jira_key`
  - `summary`
  - Roadmap sprint from `customfield_10064`
  - status
  - Launch Priority from `customfield_10561`
- Sprint range, for example `25092` through `26052`.
- Launch scope from the PMM SOP:
  - Help articles
  - Customer Support release notes

Read `references/launch-priority-materials.md` before classifying launch materials.

## Workflow

1. Build the source table:
   - Use KER Product Discovery `Roadmap` (`customfield_10064`) as the sprint/release bucket.
   - Use `Launch Priority` (`customfield_10561`) as the priority source.
   - Use Jira `status.name` as the ticket final/current status.
   - Do not infer launch priority from Jira engineering priority.
2. Filter launch candidates:
   - Include rows in the requested Roadmap range.
   - Treat `6 - Shipped & Launching` and `Done` as shipped candidates.
   - Keep non-shipped rows visible as blocked or not-ready when they have Launch Priority set.
3. Classify required materials:
   - P1: help article and Customer Support release notes.
   - P2: help article and Customer Support release notes.
   - P3: Customer Support release notes or internal update by default; help article only when user behavior changes.
   - P4 or blank: no customer-facing launch material unless the change is customer-visible and likely to need guidance.
4. Create material work items:
   - `help_article`: route to `help-article-generator`.
   - `release_notes`: route to `customer-support-release-notes-generator`.
5. Evaluation checkpoint:
   - After every English or Indonesian help article draft or update patch, run `help-article-validator`.
   - If the help-article validator returns `Revise before drafting`, run `help-article-feedback-updater`, then rerun `help-article-validator`.
   - Do not mark help articles `ready_for_review` unless both required locales return `Ready to draft`.
   - After every Customer Support release note draft, run `customer-support-release-notes-validator`.
   - If the release-note validator returns `revise`, run `customer-support-release-notes-feedback-updater`, then rerun `customer-support-release-notes-validator`.
   - Do not mark release notes `ready_for_review` unless the validator decision is `pass`.
6. Output the launch tracker:
   - `not_needed`
   - `needed`
   - `drafted`
   - `needs_revision`
   - `ready_for_review`
   - `approved`
   - `published_manual`
   - `blocked`

## Output Contract

Return a compact table with:

```text
Roadmap | Jira | Status | Launch Priority | Help Article | Release Notes | Blocker / Next Action
```

For generated material drafts, include:

```text
Material: <release_notes>
Draft: <copy>
Evaluator decision: <pass | revise | blocked>
Evaluator confidence: <0-100>
Next action: <specific owner action>
```

For generated help article drafts, include:

```text
Material: <help_article>
Locale: <en | id>
Draft or patch: <copy>
Evaluator decision: <Ready to draft | Revise before drafting | Do not draft>
Evaluator confidence: <0-100>
Evidence-based reasoning: <short bullets>
Next action: <specific owner action>
```

## Guardrails

- Customer-facing outputs are draft-first and approval-gated.
- Public Intercom publishing stays manual.
- Do not expose raw Jira descriptions, comments, customer PII, internal app names, or private implementation details.
- If Launch Priority is blank, do not invent it. Use SOP heuristics only as a `suggested_priority` and mark confidence `needs-check`.
- Changelog / What's New and WhatsApp Community messages are out of scope for this workflow. Do not draft, evaluate, or track them here.
