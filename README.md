# Agent Team

A portable Markdown agent team for making Codex act like a gated AI software development team inside any repository.

## Developer Note

This project intentionally does not include a custom runtime. The runtime is Codex itself. `AGENTS.md` tells Codex what to read, and `agent-team-protocol/` contains the durable team rules.

When maintaining this repo, keep this distinction clear:

- `AGENTS.md` is the small loader Codex sees first.
- `agent-team-protocol/` is the detailed protocol Codex reads next.
- `INSTALL.md` explains how to copy the protocol into another repo.

For Codex-style usage, this repository includes [AGENTS.md](AGENTS.md), an AI-readable loader file. Detailed rules live in `agent-team-protocol/`. Copy both `AGENTS.md` and `agent-team-protocol/` into any target repository root to make Codex follow the same team roles, gates, artifact handoff, loop rules, and release checks while you chat normally.

See [INSTALL.md](INSTALL.md) for copy/install instructions.

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

## Not A Runtime

Agent Team is a Markdown rule system for Codex, not a custom agent runtime.

It does not include:

- a server
- a database
- a queue
- vector memory
- background workers
- automatic multi-agent orchestration outside Codex

Real runtime systems are useful when you want a productized agent platform, CI bot, long-running background jobs, persistent memory, logs, evals, or automatic task execution. This project is intentionally lighter: copy `AGENTS.md` and `agent-team-protocol/` into a project, then work with Codex normally.

The design follows the roles and gates defined in this conversation:

- Lead
- Product Designer
- Project Explorer
- Architect
- Frontend Engineer
- Backend Engineer
- DevOps Engineer
- QA Engineer
- Code Reviewer
- Security Reviewer

The important idea is artifact handoff. Agents do not freely chat with each other. Each gate writes a structured artifact, and the next gate consumes only the artifacts it needs.

## Engineering Model

This protocol uses a sub-agent orchestration model with workflow gates.

In practical terms:

- The user speaks only to Lead.
- Lead acts as the orchestrator.
- Each role acts like a specialist sub-agent.
- Each specialist receives only the inputs it is allowed to use.
- Each specialist returns an artifact instead of joining a free-form group chat.
- Review roles judge evidence independently instead of trusting implementation reasoning.

When a real multi-agent runtime is available, each role should run in its own context window.

When only one Codex conversation is available, the protocol simulates sub-agent separation through `agent-team-protocol/09-context-boundaries.md`. Lead packages the allowed inputs for each role, and each role must ignore forbidden inputs such as another role's private reasoning, unrelated transcript history, or unapproved scope changes.

The team also uses Agent Team-style coordination, but only through controlled artifacts, gates, and loops:

- artifacts preserve decisions and evidence
- gates prevent skipped work
- loops send failed work back to the nearest responsible role
- QA, Code Review, and Security remain independent release checks

The result is intentionally not a free-chat agent swarm. It is closer to:

```text
Lead orchestrator
 -> isolated specialist sub-agents
 -> artifact handoff
 -> gated review
 -> repair loops
 -> final release decision
```

## Quality Rules

The protocol keeps extra rigor lightweight. It does not require a full ceremony for every small task, but it does add stronger checks where mistakes are costly:

- Lead packages each role with allowed inputs, evidence included, evidence missing, and expected output.
- Architect separates repository facts, verified external facts, assumptions, and recommendations.
- Architect records source URL, date checked, confidence, and supported decision for verified external facts.
- Engineers work in small implementation slices with checks and rollback notes.
- QA checks browser behavior, console errors, and core interactions when UI is involved and tools are available.
- Code Reviewer uses a short doubt check for the riskiest correctness claim.
- Security Reviewer marks important areas as `Reviewed`, `Not relevant`, or `Not checked`.
- Fast paths are forbidden whenever a must-ask trigger applies.
- The same blocking failure stops the loop after 2 consecutive unresolved attempts.

These rules are meant to improve independence and evidence quality without turning every task into a heavy process.

## Quick Start

Copy these into the root of any project where you want Codex to follow this team protocol:

```text
AGENTS.md
agent-team-protocol/
```

Then start a new Codex session from that project root and talk normally:

```text
Help me add login.
Review whether this branch is ready to release.
Refactor the dashboard export flow.
```

Codex should treat you as speaking to Lead, then route work through the protocol gates.

## Workflow

```text
Lead
 -> Product Designer
 -> Project Explorer
 -> Architect
 -> DevOps Engineer, early release feasibility
 -> User Approval
 -> Frontend Engineer / Backend Engineer / DevOps Engineer
 -> QA Engineer
 -> Code Reviewer
 -> Security Reviewer
 -> Lead Final
```

Implementation is blocked until user approval.

Security Reviewer is a hard release gate and can veto release.

During implementation, the protocol expects separate engineering handoffs:

```text
06a-frontend-implementation.md
06b-backend-implementation.md
06c-devops-implementation.md
06-implementation-summary.md
```

QA, Code Reviewer, and Security Reviewer read these artifacts independently.

## File Map

```text
AGENTS.md
  Loader file. Keep short. It tells Codex which protocol files to read.

agent-team-protocol/
  Main rules for gates, artifacts, approval, loops, and release policy.

agent-team-protocol/roles/
  One file per role. Add role-specific rules here, not in AGENTS.md.

agent-team-protocol/examples/
  Trial workflows for testing the protocol in a real repo.

agent-team-protocol/artifacts/
  Optional temporary task artifacts. Ignored by git unless the user explicitly asks to preserve them.

INSTALL.md
  Copy/install instructions for another repository.
```

## Ask-User Policy

The protocol should ask the user before continuing when:

- Scope or acceptance criteria are unclear.
- A product tradeoff is needed.
- A new dependency, paid service, external account, or vendor lock-in is introduced.
- Database schema, migrations, auth, permissions, or privacy behavior changes.
- Deployment, CI/CD, environment variables, secrets, cron, workers, or cloud resources change.
- A feature is deleted, a module is rewritten, or a broad refactor is needed.
- Reviewers disagree on release readiness.
- Security returns `RELEASE_OK_WITH_RISK_ACCEPTANCE` or `RELEASE_BLOCKED`.
- The task expands materially beyond the original scope or budget.

Fast path is not allowed when any of these triggers applies.

## Loop Rules

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

## Design References

This protocol is influenced by public agent architecture and engineering workflow patterns from OpenAI, Anthropic, and agent-skills:

- [OpenAI Codex Subagents](https://developers.openai.com/codex/concepts/subagents)
- [OpenAI Agents SDK Handoffs](https://openai.github.io/openai-agents-python/handoffs/)
- [Anthropic Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)
- [agent-skills: Orchestration Patterns](https://github.com/addyosmani/agent-skills/blob/main/references/orchestration-patterns.md)

The protocol combines these ideas into a repo-portable Markdown system: sub-agent style role isolation, artifact handoff, explicit gates, user approval, review independence, security veto, and recovery loops.
