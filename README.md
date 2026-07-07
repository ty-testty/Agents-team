# Agent Team

Agent Team is a portable Markdown protocol that makes Codex work more like a disciplined software team inside any repository.

You still chat with Codex normally. The difference is that Codex follows a team workflow: plan first, ask before implementation, make changes inside approved scope, then pass QA, Code Review, and Security Review before recommending release.

## What This Is

This repo gives Codex a reusable team operating manual.

In plain terms:

1. You talk to `Lead`.
2. `Lead` creates bounded task packets for specialist subagents.
3. Specialist subagents return structured artifacts.
4. QA, Code Review, and Security inspect evidence independently.
5. `Lead` summarizes the result and tells you whether release is recommended.

It is intentionally not a free-chat agent swarm. Roles do not casually talk to each other; they hand off artifacts.

```text
User
 -> Lead
 -> specialist subagents
 -> artifacts
 -> gates and recovery loops
 -> final release decision
```

## Key Terms

- `Lead`: the only role you talk to directly. Lead routes work, asks for approval, validates artifacts, and summarizes final status.
- `Subagent`: a specialist role running in its own context, such as Architect, QA Engineer, or Security Reviewer.
- `Gate`: a checkpoint that must produce an artifact or clear decision before the workflow moves on.
- `Artifact`: a structured handoff result. It records decisions, evidence, missing evidence, blockers, and next action.
- `Role Packet`: the bounded task packet Lead sends to a subagent. It lists the objective, allowed inputs, forbidden inputs, evidence, and expected output.
- `Implementation Freeze`: the point after implementation when the diff or implementation artifact is frozen so QA, Code Review, and Security can review the same thing.
- `Security Veto`: Security Reviewer can block release with `RELEASE_BLOCKED`; Lead cannot override it.
- `Runtime`: a real server/platform that runs agents automatically. This project is not that. Codex is the runtime here.

## Use Cases

Agent Team is best suited for personal development and small-team software work where you want AI help without losing order, review quality, or release safety.

Good fits:

- personal projects and small-team repositories
- MVPs, prototypes, small SaaS products, and web apps
- adding features to an existing project
- bug fixes and focused refactors
- frontend UI/UX improvements
- API, auth, permission, and privacy-related changes
- release-readiness checks with QA, Code Review, and Security Review

The ideal task size is a feature, fix, or refactor that can be completed in roughly 30 minutes to 2 days. Larger work should be split by Lead into smaller gated tasks.

## Workflow

For normal software work, the default flow is:

```text
Lead
 -> Product Designer
 -> Project Explorer
 -> Architect
 -> DevOps Engineer, early read-only release feasibility
 -> User Approval
 -> Frontend Engineer / Backend Engineer / DevOps Engineer
 -> Implementation Freeze
 -> QA Engineer / Code Reviewer / Security Reviewer
 -> Lead Final
```

Important rules:

- Implementation is blocked until you explicitly approve it.
- Fast paths can shorten planning, but they cannot skip user approval.
- QA, Code Review, and Security can run in parallel after Implementation Freeze.
- Release status must wait for all required review artifacts.
- Security Reviewer is a hard release gate and can veto release.

## Roles

- `Product Designer`: defines user goal, scope, acceptance criteria, UX flow, and UI states.
- `Project Explorer`: reads the repository and reports facts, existing patterns, commands, dependencies, and constraints.
- `Architect`: creates the technical plan, contracts, tradeoffs, test strategy, rollback plan, and verifies current external facts when needed.
- `Frontend Engineer`: implements approved UI, client behavior, styling, and frontend tests.
- `Backend Engineer`: implements approved APIs, server logic, data handling, permissions, and backend tests.
- `DevOps Engineer`: checks release/runtime feasibility early; implements CI/CD, deployment, env, and runtime changes only after approval.
- `QA Engineer`: verifies behavior and UX using evidence. QA is read-only and asks Lead for missing test/build/browser evidence.
- `Code Reviewer`: independently reviews correctness, maintainability, regressions, and missing tests.
- `Security Reviewer`: independently reviews auth, permissions, data, secrets, dependencies, deployment risk, and release security.

## Subagent Mode

Agent Team uses one execution model: Subagent Mode.

```text
Lead dispatches each role with a bounded Role Packet.
Lead validates the Role Packet before dispatch.
The role runs as a specialist subagent.
Each role returns an artifact.
Lead validates the artifact before a gate can pass.
Gates and loops consume artifacts, not private reasoning.
```

Codex subagent configuration lives in:

```text
.codex/config.toml
.codex/agents/*.toml
.codex/agents/README.md
```

Lead is not a subagent. Lead is the parent orchestrator.

If Codex subagent threads are unavailable, Lead must report `SUBAGENT_UNAVAILABLE` and ask whether to enable subagents, continue with reduced independence for that task, or cancel. Reduced-independence execution is an explicit waiver, not normal Subagent Mode.

## Quick Start

Copy these files into the root of any project where you want Codex to follow this workflow:

```text
AGENTS.md
agent-team-protocol/
.codex/
```

Then start a new Codex session from that project root and talk normally:

```text
Help me add login.
Review whether this branch is ready to release.
Refactor the dashboard export flow.
```

Codex should treat you as speaking to `Lead`, then route the work through the protocol.

See [INSTALL.md](INSTALL.md) for copy/install instructions.

## Quality Rules

The protocol adds checks where mistakes are costly, without turning every small task into a heavy ceremony.

- Lead packages each role with allowed inputs, forbidden inputs, evidence included, evidence missing, and expected output.
- Role Packets and returned artifacts must pass validity checks before Lead can dispatch or mark a gate passed.
- Context budget reduces duplicated context without reducing gates, reviewer independence, approval, or release checks.
- Architect separates repository facts, verified external facts, assumptions, and recommendations.
- Engineers work in small implementation slices with checks and rollback notes.
- QA verifies evidence rather than trusting implementation claims.
- Code Reviewer uses a short doubt check for the riskiest correctness claim.
- Security Reviewer marks important areas as `Reviewed`, `Not relevant`, or `Not checked`.
- The same blocking failure stops the loop after 2 consecutive unresolved attempts.

## Not A Runtime

Agent Team is a Markdown rule system for Codex, not a custom agent runtime.

It does not include:

- a server
- a database
- a queue
- vector memory
- background workers
- automatic multi-agent orchestration outside Codex

Real runtime systems are useful when you want a productized agent platform, CI bot, long-running background jobs, persistent memory, logs, evals, or automatic task execution. This project is intentionally lighter: copy `AGENTS.md`, `agent-team-protocol/`, and `.codex/` into a project, then work with Codex normally.

## File Map

```text
AGENTS.md
  Loader file. Keep short. It tells Codex which protocol files to read.

agent-team-protocol/
  Main rules for gates, artifacts, approval, loops, boundaries, and release policy.

agent-team-protocol/roles/
  One file per role. Add role-specific rules here, not in AGENTS.md.

agent-team-protocol/examples/
  Trial workflows for testing the protocol in a real repo.

.agent-team/runs/
  Optional temporary role packets, artifacts, logs, and patches. Ignored by git unless the user explicitly asks to preserve them.

.codex/
  Codex custom agent definitions. Includes `.codex/agents/README.md`, which maps agent names to role specs.

INSTALL.md
  Copy/install instructions for another repository.
```

## Ask-User Policy

Codex may analyze, inspect, and propose a plan, but implementation requires explicit user permission.

The protocol asks before continuing when:

- scope or acceptance criteria are unclear
- a product tradeoff is needed
- a new dependency, paid service, external account, or vendor lock-in is introduced
- database schema, migrations, auth, permissions, or privacy behavior changes
- deployment, CI/CD, environment variables, secrets, cron, workers, or cloud resources change
- a feature is deleted, a module is rewritten, or a broad refactor is needed
- reviewers disagree on release readiness
- Security returns `RELEASE_OK_WITH_RISK_ACCEPTANCE` or `RELEASE_BLOCKED`
- the task expands materially beyond the original scope or budget

Fast path is not allowed when any of these triggers applies.

## Loop Rules

The workflow does not restart from zero after every failure. Lead sends the work back to the nearest responsible gate.

```text
QA failed
 -> responsible engineer
 -> QA rerun
 -> Code Review and Security when relevant

Code Review blocked
 -> responsible engineer
 -> focused QA
 -> Code Reviewer rereview

Security blocked
 -> Architect or responsible engineer
 -> QA
 -> Code Review
 -> Security rereview

Scope changed
 -> Product Gate
 -> regenerate downstream artifacts
 -> user approval

Same blocking failure repeated 2 consecutive times
 -> stop
 -> Lead reports what failed, what was tried, remaining options, and recommended next step
 -> third attempt requires explicit user approval
```

## Maintainer Notes

When maintaining this repo, keep this distinction clear:

- `AGENTS.md` is the small loader Codex sees first.
- `agent-team-protocol/` contains durable protocol rules.
- `.codex/` contains Codex custom agent definitions for Subagent Mode.
- `INSTALL.md` explains how to copy the protocol into another repo.

Do not duplicate the full protocol in `AGENTS.md`; duplication makes future edits drift.

## Design References

This protocol is influenced by public agent architecture and engineering workflow patterns from OpenAI, Anthropic, and agent-skills:

- [OpenAI Codex Subagents Setup](https://developers.openai.com/codex/subagents)
- [OpenAI Codex Subagents](https://developers.openai.com/codex/concepts/subagents)
- [OpenAI Agents SDK Handoffs](https://openai.github.io/openai-agents-python/handoffs/)
- [Anthropic Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)
- [agent-skills: Orchestration Patterns](https://github.com/addyosmani/agent-skills/blob/main/references/orchestration-patterns.md)

The protocol combines these ideas into a repo-portable Markdown system: sub-agent style role isolation, artifact handoff, explicit gates, user approval, review independence, security veto, and recovery loops.
