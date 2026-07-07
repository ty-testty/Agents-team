# Context Budget

## Developer Note

This file controls context size without weakening the Agent Team design. It must preserve gates, role separation, artifact validity, user approval, release review, and Security Reviewer veto power.

The goal is not to use fewer roles by default. The goal is to give each role only the context it needs.

## Principle

Preserve gates and role separation. Reduce duplicated context.

Context budget rules must never be used to:

- skip required user approval
- skip required QA, Code Review, or Security Review
- weaken Security Reviewer veto power
- merge independent review roles into one shared context
- hide missing evidence
- treat summaries as proof when primary evidence is available

## Lead Context Budget Duty

Lead should keep Role Packets small, explicit, and evidence-based.

Lead must not paste the full transcript into Role Packets by default. Use references to artifacts, logs, diffs, screenshots, and command output when those references are enough.

Lead should include excerpts only when:

- the exact text is needed for the role's decision
- the source is hard to locate from a reference
- the excerpt prevents ambiguity
- the excerpt is shorter and clearer than asking the role to inspect the full source

Lead should prefer:

- artifact references over copied artifact bodies
- changed file lists over full diffs when the role only needs scope
- focused diff excerpts over full diffs when the role is checking one area
- command summaries plus log references over full logs
- screenshots or browser evidence only when UI behavior matters

## Subagent Read Budget

Subagents should read only:

- their own role file
- the relevant protocol sections
- the bounded Role Packet
- approved artifacts needed for the task
- required repository evidence
- required command, test, screenshot, log, or diff evidence

Subagents should not read broad transcript history, unrelated artifacts, or other roles' private reasoning.

If a subagent needs more evidence, it should return `NEEDS_MORE_EVIDENCE` or `BLOCKED` instead of expanding context on its own.

## Role Packet Budget

Role Packets should be reference-first:

```text
Artifact Reference:
Evidence Reference:
Diff Reference:
Log Reference:
Screenshot Reference:
Exact Excerpts Included:
Evidence Missing:
```

Use exact excerpts only when necessary. If a packet contains a long pasted section, Lead should state why the excerpt is needed.

The Role Packet Validity Rule in `agent-team-protocol/06-output-templates.md` still applies. A smaller packet is not valid if it omits required scope, evidence, missing evidence, or stop conditions.

## Artifact Budget

Artifacts should be decision-first and compact:

```text
Decision:
Evidence:
Assumptions:
Missing Or Unverified Evidence:
Blockers:
Next Action:
```

Artifacts should summarize findings and reference evidence. They should not include long private deliberations or duplicate broad logs unless those logs are the evidence being reviewed.

The Artifact Validity Rule in `agent-team-protocol/06-output-templates.md` still applies. A compact artifact is not valid if it hides uncertainty, blockers, or missing evidence.

## Spawn Policy

Use the smallest role set that can preserve correctness and release safety.

For tiny, low-risk, text-only tasks, Lead may use fewer subagents when the normal fast-path rules allow it.

For medium-risk or high-risk tasks, release-relevant changes, security-sensitive changes, broad implementation changes, dependency changes, data changes, auth changes, deployment changes, or user-facing behavior changes, Lead should preserve the normal gate sequence and full release review.

After implementation freeze, QA Engineer, Code Reviewer, and Security Reviewer may run in parallel to reduce elapsed time, but their context packets must remain independent and evidence-based.

## Stop Conditions

Lead should stop and ask the user when reducing context would make a role's decision unreliable.

Examples:

- evidence is too large to summarize safely
- the role needs primary source review but only a summary is available
- a reviewer cannot inspect the diff, logs, tests, or screenshots needed for its decision
- the same missing evidence blocks the workflow twice
- context reduction would require skipping a gate

When blocked by context budget, Lead should report what evidence is missing, which role needs it, and the smallest useful next step.
