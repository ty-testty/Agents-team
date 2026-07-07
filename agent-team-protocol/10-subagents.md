# Subagents

## Developer Note

This file defines Subagent Mode for the Agent Team. It maps roles, gates, artifacts, and loops onto specialist subagents. Codex subagent threads are the primary execution surface, and `.codex/agents/` contains the project-scoped custom agent definitions.

## Core Rule

Lead remains the only user-facing entry point.

Subagents are delegated workers. They do not chat freely with each other, do not bypass gates, and do not make final release decisions outside the normal workflow.

Only Lead may dispatch subagents. Subagents must not spawn, invoke, or route work to other subagents.

```text
User
 -> Lead
 -> specialist subagents
 -> artifacts returned to Lead
 -> gates, loops, and final release decision
```

## Execution Surface

Agent Team has one Subagent Mode.

Use Codex subagent threads for non-trivial role work.

If the environment cannot spawn Codex subagent threads, Lead must not silently emulate full agent independence in one shared context. Lead must return `SUBAGENT_UNAVAILABLE` and ask the user whether to:

- enable or install Codex subagent support
- continue with reduced independence for this task
- cancel the task

Continuing with reduced independence is an explicit user waiver, not a normal execution path.

If subagent dispatch would materially increase cost, runtime, tool use, or file-write risk, Lead should ask for confirmation before expanding the workflow.

## Required Files

Codex subagent configuration lives in:

```text
.codex/config.toml
.codex/agents/*.toml
.codex/agents/README.md
```

Durable team rules still live in:

```text
agent-team-protocol/
```

If `.codex/agents/` is missing or unsupported, mark the workflow `SUBAGENT_UNAVAILABLE`. Do not continue as if independent subagents are active.

Use `.codex/agents/README.md` as the human-readable mapping between TOML files, Codex agent `name` fields, and role spec files. Codex dispatch is keyed by the TOML `name` field, so filename, `name`, and role spec must stay aligned.

## Agent Mapping

| Team role | Codex custom agent | Default sandbox | Typical use |
|---|---|---:|---|
| Product Designer | `product_designer` | read-only | scope, acceptance criteria, UX flow |
| Project Explorer | `project_explorer` | read-only | repo facts, affected files, existing patterns |
| Architect | `architect` | read-only | design, contracts, external fact verification |
| Frontend Engineer | `frontend_engineer` | workspace-write | approved frontend implementation |
| Backend Engineer | `backend_engineer` | workspace-write | approved backend implementation |
| DevOps Engineer | `devops_engineer` | workspace-write | approved runtime, CI/CD, deployment changes |
| QA Engineer | `qa_engineer` | read-only | behavior, UX/UI, verification evidence |
| Code Reviewer | `code_reviewer` | read-only | correctness, maintainability, test gaps |
| Security Reviewer | `security_reviewer` | read-only | release-blocking security review |

Lead is not a subagent. Lead is the parent agent coordinating the workflow.

DevOps Engineer has workspace-write capability for approved implementation work, but the early Release Feasibility Gate is a read-only review phase. Lead must encode that phase boundary in the Role Packet.

QA Engineer is read-only. QA verifies provided evidence and requests missing evidence through Lead when verification requires commands, servers, downloads, or other execution actions.

## Dispatch Requirements

Before dispatching a subagent, Lead must prepare a bounded Role Packet with:

- role name
- task objective
- allowed inputs
- forbidden inputs
- approved scope
- evidence included
- evidence missing
- required output artifact
- artifact location or transcript location
- stop conditions

The Role Packet must satisfy the Role Packet Validity Rule in `agent-team-protocol/06-output-templates.md`. If the packet is incomplete, Lead must return `PACKET_INCOMPLETE` and repair the packet before dispatch.

The subagent should return only the requested artifact or concise findings. Lead must not paste raw intermediate logs, private reasoning, or broad transcript history back into the main context unless they are necessary evidence.

Use `agent-team-protocol/11-context-budget.md` when deciding how much transcript, artifact, diff, log, screenshot, or command evidence to include in a Role Packet.

## Dispatch Sequence

For each required gate:

1. Lead selects the correct subagent from the mapping.
2. Lead prepares a bounded Role Packet.
3. Lead validates the Role Packet.
4. Lead dispatches the subagent in a separate Codex subagent thread.
5. The subagent returns only its artifact or concise findings.
6. Lead validates the returned artifact against the Artifact Validity Rule.
7. Lead routes the artifact to the next gate or loop.

Lead must not replace a required specialist subagent with Lead's own judgment.

## Fan-Out Review

After implementation is complete, Lead should freeze the implementation artifact and dispatch QA Engineer, Code Reviewer, and Security Reviewer in parallel when the change is ready for release review.

The fan-out review packet should include:

- approved scope
- architecture and implementation artifacts
- frozen diff or changed files
- command output, screenshots, logs, tests, or browser evidence
- review checklists
- release policy

During fan-out review:

- reviewers are read-only
- reviewers do not call each other
- reviewers do not call other subagents
- reviewers do not rely on implementation private reasoning
- Lead waits for all required review artifacts before final release status

If implementation changes after any reviewer starts, Lead must mark the affected review artifacts stale and rerun the relevant reviewers.

## Parallelism

Parallel subagents are preferred for independent, read-heavy work:

- Project Explorer maps code paths while Architect verifies external facts.
- QA, Code Review, and Security Review independently inspect the same completed diff.
- Multiple reviewers inspect separate packages, services, or files.

Parallel subagents require caution when writing files:

- Only one implementation subagent should own a file or tightly coupled area at a time.
- Frontend, Backend, and DevOps implementation may run in parallel only when Lead can clearly separate ownership and the user has approved that split.
- If ownership overlaps, use sequential implementation instead.

## Write Safety

Implementation subagents must obey the same approval policy as every other role.

They must stop and return to Lead if:

- implementation approval is missing
- scope changes
- a new dependency is needed
- auth, permissions, data, privacy, deployment, CI/CD, secrets, migrations, workers, or cloud resources change
- another implementation subagent owns the same file or contract
- tests expose the same blocking failure twice

## Review Independence

QA Engineer, Code Reviewer, and Security Reviewer should be strong candidates for Codex subagent threads because their value depends on independent judgment.

Review subagents should receive evidence, not persuasion:

- approved scope
- relevant artifacts
- diff or changed files
- command output, screenshots, logs, tests, or browser evidence
- checklists and release policy

Review subagents should not receive:

- implementation private reasoning
- unrelated chat history
- engineer self-justification
- unapproved scope changes

Review subagents must not dispatch other subagents. If a reviewer needs additional evidence, it returns `NEEDS_MORE_EVIDENCE` or `BLOCKED` to Lead.

## Output Rule

Subagent output must become an artifact or be summarized into one. Downstream gates consume the artifact, not the subagent's private reasoning.

If a subagent fails, times out, or returns insufficient evidence, Lead must mark the relevant gate as `NOT_RUN`, `BLOCKED`, or `NEEDS_MORE_EVIDENCE` instead of pretending the review passed.

## Cost And Scope Control

Subagents can improve independence, context hygiene, and parallel review quality, but they cost more tokens and coordination.

Use parallel subagents when the benefit is clear:

- medium-risk or high-risk changes
- release readiness review
- security-sensitive changes
- broad code exploration
- multiple independent review perspectives
- large logs, long documents, or noisy test output

For tiny, text-only, or low-risk work, Lead may use the smallest relevant subagent set, but any required specialist judgment still belongs to the appropriate subagent.

Context budget can reduce duplicated context, but it must not reduce required gates, user approval, reviewer independence, or release checks.
