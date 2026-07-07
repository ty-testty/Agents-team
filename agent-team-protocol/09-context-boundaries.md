# Context Boundaries

## Developer Note

This file turns subagent independence into an operational rule. Every role works from a bounded Role Packet, returns an artifact, and avoids private reasoning from other roles. Subagent work should run in separate Codex subagent context windows.

## Rule

Each role must work from a bounded input packet.

The packet must contain only:

- the current task objective
- the relevant approved artifacts
- the relevant repository evidence
- the relevant command or test evidence
- the protocol rules needed by that role
- the expected output artifact

The packet must not contain:

- another role's private reasoning
- raw hidden chain-of-thought
- persuasive commentary from the role being reviewed
- unrelated transcript history
- unapproved scope changes
- unverifiable claims used as evidence

## Subagent Execution

For non-trivial role work:

- start each role in a fresh context
- provide only the role's allowed inputs
- return only the role's output artifact
- do not copy private reasoning between contexts

Use `agent-team-protocol/10-subagents.md` for subagent dispatch rules.

Use `agent-team-protocol/11-context-budget.md` to keep packets small without weakening role separation, gates, or review requirements.

If Codex subagent threads are unavailable:

- Lead must mark the workflow `SUBAGENT_UNAVAILABLE`
- Lead must not silently continue as if independence is preserved
- Lead must ask the user whether to enable subagents, continue with reduced independence, or cancel
- if the user explicitly accepts reduced independence, Lead must disclose that QA, Code Review, and Security are no longer fully independent for that run

If a role cannot avoid contaminated context, Lead must either summarize a sanitized artifact packet or ask the user whether to continue with reduced independence.

## Lead Packaging Duty

Before dispatching a role, Lead should prepare a role packet:

```text
Role:
Objective:
Allowed Inputs:
Forbidden Inputs:
Approved Scope:
Evidence Included:
Evidence Missing:
Required Output Artifact:
Artifact Location:
Stop Conditions:
Known Unknowns:
```

Lead may summarize prior work, but the summary must be evidence-based and must not include private reasoning.

If the transcript or repository context is large, Lead should provide a short artifact digest instead of copying broad history. The digest should separate facts, decisions, assumptions, and open questions.

If evidence is missing and the missing evidence could change correctness, safety, scope, or release readiness, the role must return `BLOCKED` or ask Lead for a sanitized packet.

## Boundary Violation

If a role receives forbidden input, it must:

1. ignore the forbidden input
2. name the boundary issue briefly in its artifact
3. continue only if the remaining allowed inputs are sufficient
4. ask Lead for a sanitized packet if the remaining inputs are not sufficient

## Reviewer Independence

QA Engineer, Code Reviewer, and Security Reviewer must treat implementation claims as claims, not proof.

For medium-risk and high-risk tasks, reviewers should inspect artifacts, diff, command output, screenshots, logs, and tests before relying on implementation summaries.

Review evidence should come from:

- code or diff
- tests, builds, lint, typecheck, screenshots, or logs
- approved product and architecture artifacts
- project rules and security requirements

Review evidence should not come from:

- engineer confidence
- private implementation reasoning
- "should work" statements
- summaries without evidence

## Lightweight Doubt Check

For high-impact claims, planning decisions, and release decisions, use this short check:

```text
Claim:
Evidence:
What could make this wrong?
Cheapest verification:
Decision:
```

Use it when a decision involves security, auth, data loss, migrations, new dependencies, deployment behavior, external APIs, or broad refactors. Do not use it for trivial text-only changes unless risk is unclear.

## Artifact Privacy

Artifacts should contain decisions, evidence, risks, and next steps.

Artifacts should not contain hidden reasoning transcripts or long private deliberations. The goal is to make handoff inspectable without blending agent contexts.
