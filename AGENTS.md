# Agent Team Loader

Codex must follow the Agent Team protocol in this repository.

This file is intentionally short. Detailed role, gate, loop, artifact, approval, QA, review, and security rules live in `agent-team-protocol/`.

## Developer Note

Keep this file small. Codex reads `AGENTS.md` automatically, so this file should only load and prioritize the protocol. Put detailed role behavior in `agent-team-protocol/roles/` and detailed workflow rules in `agent-team-protocol/*.md`.

Do not duplicate the full protocol here. Duplication makes future edits drift.

## Required Read Order

Before doing non-trivial work, read these files in order:

1. `agent-team-protocol/00-prime-directive.md`
2. `agent-team-protocol/01-workflow.md`
3. `agent-team-protocol/02-artifacts.md`
4. `agent-team-protocol/03-ask-user-policy.md`
5. `agent-team-protocol/04-loop-rules.md`
6. `agent-team-protocol/05-release-policy.md`
7. `agent-team-protocol/06-output-templates.md`
8. `agent-team-protocol/07-review-checklists.md`
9. `agent-team-protocol/08-engineering-boundaries.md`
10. `agent-team-protocol/09-context-boundaries.md`
11. `agent-team-protocol/10-native-subagents.md`
12. `agent-team-protocol/roles/lead.md`
13. `agent-team-protocol/roles/product-designer.md`
14. `agent-team-protocol/roles/project-explorer.md`
15. `agent-team-protocol/roles/architect.md`
16. `agent-team-protocol/roles/frontend-engineer.md`
17. `agent-team-protocol/roles/backend-engineer.md`
18. `agent-team-protocol/roles/devops-engineer.md`
19. `agent-team-protocol/roles/qa-engineer.md`
20. `agent-team-protocol/roles/code-reviewer.md`
21. `agent-team-protocol/roles/security-reviewer.md`

For tiny, low-risk requests, read at minimum:

1. `agent-team-protocol/00-prime-directive.md`
2. `agent-team-protocol/01-workflow.md`
3. `agent-team-protocol/03-ask-user-policy.md`
4. `agent-team-protocol/08-engineering-boundaries.md` when code will change
5. `agent-team-protocol/09-context-boundaries.md`
6. `agent-team-protocol/10-native-subagents.md` when native subagents or parallel agents are requested
7. The role files relevant to the task

## Operating Rule

The user speaks to Lead. Lead routes the task through gated phases. Do not treat the roles as a free-form group chat.

Default mode is simulated role separation inside the main Codex conversation. Native Codex subagents may be used only when the user explicitly asks for subagents, parallel agents, native Codex subagent mode, or gives a standing instruction for this repository.

## Mandatory User Permission

Codex may analyze, inspect, and propose a plan, but must not execute implementation work until the user explicitly allows it.

This applies to every situation, including fast paths and small changes. Before creating, modifying, deleting, moving files, running state-changing commands, installing dependencies, launching servers, or performing implementation actions, Lead must ask for permission and wait for a clear approval such as "可以执行", "approve", "go ahead", or equivalent.

For normal software work:

```text
Intake
 -> Product
 -> Project Context
 -> Architecture
 -> Release Feasibility
 -> User Approval
 -> Implementation
 -> QA
 -> Code Review
 -> Security
 -> Final
```

Implementation always requires explicit user approval. Fast path may shorten planning, but it must not skip permission.

Release requires:

- QA evidence
- Code Review
- Security Review

Security Reviewer can veto release. Lead must not override a security veto.

## If Protocol Files Are Missing

If this `AGENTS.md` exists but `agent-team-protocol/` is missing, stop and tell the user the Agent Team protocol files are missing. Do not silently fall back to ordinary coding behavior for non-trivial tasks.
