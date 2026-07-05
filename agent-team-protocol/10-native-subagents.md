# Native Codex Subagents

## Developer Note

This file defines the native Codex subagent execution layer for the Agent Team. It does not replace the Markdown protocol. It maps the existing roles, gates, artifacts, and loops onto Codex custom agent threads when native subagents are explicitly enabled for a task.

## Core Rule

Lead remains the only user-facing entry point.

Native subagents are delegated workers. They do not chat freely with each other, do not bypass gates, and do not make final release decisions outside the normal workflow.

```text
User
 -> Lead
 -> native Codex subagent threads when approved
 -> artifacts returned to Lead
 -> gates, loops, and final release decision
```

## Activation

Default mode is simulated role separation inside the main Codex conversation.

Native Codex subagent mode may be used only when:

- the user explicitly asks for subagents, parallel agents, native Codex subagent mode, or equivalent wording for the current task
- or the user gives a standing instruction to use native subagents for this repository

If native subagents would materially increase cost, runtime, tool use, or file-write risk, Lead should ask for confirmation even if native mode is available.

## Required Files

Native Codex subagent configuration lives in:

```text
.codex/config.toml
.codex/agents/*.toml
```

Durable team rules still live in:

```text
agent-team-protocol/
```

If `.codex/agents/` is missing or unsupported, continue in simulated mode using `09-context-boundaries.md`.

## Agent Mapping

| Team role | Native agent | Default sandbox | Typical use |
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

Lead is not a native subagent by default. Lead is the parent agent coordinating the workflow.

## Dispatch Requirements

Before dispatching a native subagent, Lead must prepare a bounded Role Packet with:

- role name
- task objective
- dispatch mode: `SIMULATED` or `NATIVE_SUBAGENT`
- allowed inputs
- forbidden inputs
- approved scope
- evidence included
- evidence missing
- required output artifact
- artifact location or transcript location
- stop conditions

The subagent should return only the requested artifact or concise findings. Lead must not paste raw intermediate logs, private reasoning, or broad transcript history back into the main context unless they are necessary evidence.

## Parallelism

Parallel native subagents are preferred for independent, read-heavy work:

- Project Explorer maps code paths while Architect verifies external facts.
- QA, Code Review, and Security Review independently inspect the same completed diff.
- Multiple reviewers inspect separate packages, services, or files.

Parallel native subagents require caution when writing files:

- Only one implementation subagent should own a file or tightly coupled area at a time.
- Frontend, Backend, and DevOps implementation may run in parallel only when Lead can clearly separate ownership and the user has approved that split.
- If ownership overlaps, use sequential implementation instead.

## Write Safety

Native implementation subagents must obey the same approval policy as simulated roles.

They must stop and return to Lead if:

- implementation approval is missing
- scope changes
- a new dependency is needed
- auth, permissions, data, privacy, deployment, CI/CD, secrets, migrations, workers, or cloud resources change
- another implementation subagent owns the same file or contract
- tests expose the same blocking failure twice

## Review Independence

QA Engineer, Code Reviewer, and Security Reviewer should be strong candidates for native subagents because their value depends on independent judgment.

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

## Output Rule

Native subagent output must become an artifact or be summarized into one. Downstream gates consume the artifact, not the subagent's private reasoning.

If a native subagent fails, times out, or returns insufficient evidence, Lead must mark the relevant gate as `NOT_RUN`, `BLOCKED`, or `NEEDS_MORE_EVIDENCE` instead of pretending the review passed.

## Cost And Scope Control

Native subagents can improve independence, context hygiene, and parallel review quality, but they cost more tokens and coordination.

Use native mode when the benefit is clear:

- medium-risk or high-risk changes
- release readiness review
- security-sensitive changes
- broad code exploration
- multiple independent review perspectives
- large logs, long documents, or noisy test output

Prefer simulated mode when:

- the task is tiny
- the change is text-only and low risk
- user approval is missing
- the same result can be achieved with one concise role phase
