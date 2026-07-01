# Agent Team Protocol

A portable Markdown protocol for making Codex act like a gated AI software development team inside any repository.

## Developer Note

This project intentionally does not include a custom runtime anymore. The runtime is Codex itself. `AGENTS.md` tells Codex what to read, and `agent-team-protocol/` contains the durable team rules.

When maintaining this repo, keep this distinction clear:

- `AGENTS.md` is the small loader Codex sees first.
- `agent-team-protocol/` is the detailed protocol Codex reads next.
- `INSTALL.md` explains how to copy the protocol into another repo.

For Codex-style usage, this repository includes [AGENTS.md](AGENTS.md), an AI-readable loader file. Detailed rules live in `agent-team-protocol/`. Copy both `AGENTS.md` and `agent-team-protocol/` into any target repository root to make Codex follow the same team roles, gates, artifact handoff, loop rules, and release checks while you chat normally.

See [INSTALL.md](INSTALL.md) for copy/install instructions.

The design follows the roles and gates defined in this conversation:

- Lead
- Product Designer
- Project Explorer
- Research Architect
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
 -> Research Architect
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
 -> Research Architect or responsible engineer
 -> QA
 -> Code Review
 -> Security rereview

Scope changed
 -> Product Gate
 -> regenerate downstream artifacts
```

## Design References

This protocol is influenced by public agent architecture patterns from OpenAI and Anthropic:

- [OpenAI Codex Subagents](https://developers.openai.com/codex/concepts/subagents)
- [OpenAI Agents SDK Handoffs](https://openai.github.io/openai-agents-python/handoffs/)
- [Anthropic Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)

The protocol combines these ideas into a repo-portable Markdown system: sub-agent style role isolation, artifact handoff, explicit gates, user approval, review independence, security veto, and recovery loops.
