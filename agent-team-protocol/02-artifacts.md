# Artifact Handoff

## Developer Note

Artifacts are the contract between roles. They are not bureaucracy; they prevent reviewer contamination, preserve decisions, and make failed work resumable.

Agents should not freely chat with each other. They should hand off structured artifacts.

Artifact handoff gives the team:

- auditability
- resumability
- clearer gates
- less context contamination
- easier rereview after fixes
- comparable options
- a visible task timeline

## Rule

Each role outputs a structured artifact. Downstream roles read only the artifacts they need and only the inputs allowed by `agent-team-protocol/09-context-boundaries.md`. Review roles must not rely on engineering private reasoning.

## Storage And Lifecycle

Preferred file location when artifacts are written:

```text
agent-team-protocol/artifacts/<task-id>/
```

Artifacts are temporary by default and should not be committed unless the user explicitly asks to preserve them or the task requires an audit trail. This repository ignores `agent-team-protocol/artifacts/` by default.

If the user asks to commit artifacts, say that the directory is ignored by default and either intentionally force-add the specific artifact files or copy the final artifact summary to a non-ignored documentation location.

Archive artifacts only for high-risk, release-relevant, security-sensitive, or user-requested tasks. A typical archive path is:

```text
agent-team-protocol/artifacts/archive/<task-id>/
```

For small tasks, temporary experiments, typo fixes, or low-risk changes, delete or leave ignored artifacts after the task is complete.

## Preferred Artifact Fields

Use these fields when practical:

- Decision
- Inputs
- Findings
- Evidence
- Risks
- Blocking Issues
- Non-blocking Issues
- Required Next Step

## Review Inputs

QA Engineer, Code Reviewer, and Security Reviewer should read:

- original user request
- approved product brief
- approved architecture plan
- implementation artifacts or diff
- test logs, screenshots, or runtime evidence
- project rules

They should not depend on:

- private implementation reasoning
- unverifiable claims
- vague summaries without evidence
- unrelated transcript history
- unapproved scope changes

## Artifact Names

Recommended artifact names:

```text
00-intake.md
01-product-brief.md
02-project-context.md
03-architecture-plan.md
04-devops-feasibility.md
05-approval-request.md
06a-frontend-implementation.md
06b-backend-implementation.md
06c-devops-implementation.md
06-implementation-summary.md
07-qa-report.md
08-code-review.md
09-security-review.md
10-final-summary.md
```

When working only in chat and not writing artifact files, keep the same sections in the response or internal working notes.
