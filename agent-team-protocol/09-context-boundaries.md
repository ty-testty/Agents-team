# Context Boundaries

## Developer Note

This file turns "independent agents" into an operational rule. If real subagents are available and approved for the task, use separate context windows. If only one AI context is available, or native subagents were not requested, simulate the boundary by role packets, allowed inputs, forbidden inputs, and artifact-only handoff.

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

## Runtime Mode

When real subagents or a multi-agent runtime are available and approved for the task:

- start each role in a fresh context
- provide only the role's allowed inputs
- return only the role's output artifact
- do not copy private reasoning between contexts

Use `agent-team-protocol/10-native-subagents.md` for native Codex subagent dispatch rules.

## Simulated Mode

When only one AI context is available:

- announce the active role phase
- read only the allowed inputs for that role
- ignore forbidden inputs even if they exist in the transcript
- produce the role's artifact before moving to the next role
- do not let implementation self-justification influence QA, Code Review, or Security Review

If a role cannot avoid contaminated context, Lead must either summarize a sanitized artifact packet or ask the user whether to continue with reduced independence.

## Lead Packaging Duty

Before dispatching a role, Lead should prepare a role packet:

```text
Role:
Objective:
Dispatch Mode:
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
