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
