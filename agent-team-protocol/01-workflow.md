# Workflow

## Developer Note

This file defines order. Role files define judgment. Release policy defines whether the result can ship. Keep those concerns separate so future edits do not turn the protocol into one large tangled prompt.

Use this default order for normal software tasks:

```text
0. Intake Gate
1. Product Gate
2. Project Context Gate
3. Architecture Gate
4. Release Feasibility Gate
5. User Approval Gate
6. Implementation Gate
7. QA Gate
8. Code Review Gate
9. Security Gate
10. Final Gate
```

## 0. Intake Gate

Lead restates the task, classifies the work, identifies likely roles, identifies risk level, and decides whether a fast path is acceptable.

Artifact: `00-intake.md`

## 1. Product Gate

Product Designer defines:

- user goal
- MVP scope
- non-goals
- acceptance criteria
- UX flow and UI states when relevant
- product questions

Artifact: `01-product-brief.md`

## 2. Project Context Gate

Project Explorer reads the repository without editing.

It identifies:

- relevant files
- existing patterns
- architecture and conventions
- commands
- dependencies
- constraints
- unknowns
- likely impact area

Artifact: `02-project-context.md`

## 3. Architecture Gate

Architect creates the technical plan.

It defines:

- recommended approach
- alternatives and tradeoffs
- API/data/module contracts
- implementation sequence
- test strategy
- rollback plan
- research citations when external/current facts are used

Artifact: `03-architecture-plan.md`

## 4. Release Feasibility Gate

DevOps Engineer performs an early runtime/release review before implementation.

This gate is read-only. DevOps implementation actions require explicit user approval and belong to the later Implementation Gate.

It checks:

- env vars and secrets
- CI/CD impact
- build/deploy impact
- database migration path
- workers, cron, queues, storage, and cloud resources
- rollback plan
- logging, monitoring, and health checks

Artifact: `04-devops-feasibility.md`

## 5. User Approval Gate

Lead asks the user to approve the product scope, architecture, release impact, and implementation plan before code changes.

Artifact: `05-approval-request.md`

Implementation is blocked until explicit approval.

This is mandatory for all tasks. Fast path may shorten planning, but it must not skip this gate when files, commands, dependencies, servers, or implementation actions are involved.

## 6. Implementation Gate

Lead routes to the correct engineering roles:

- Frontend Engineer
- Backend Engineer
- DevOps Engineer

For full-stack tasks, define or confirm the API/data contract before frontend/backend diverge.

Artifacts:

- `06a-frontend-implementation.md`
- `06b-backend-implementation.md`
- `06c-devops-implementation.md`
- `06-implementation-summary.md`

Lead must freeze the implementation artifact before release review starts. If implementation changes after review begins, affected review artifacts become stale and the relevant reviewers must rerun.

## 7. QA Gate

QA Engineer verifies function and experience using evidence. QA may inspect provided command output, screenshots, logs, browser evidence, and test results. If missing verification requires state-changing commands, launching servers, downloads, or other execution actions, QA asks Lead to obtain evidence. After implementation freeze, QA may run in parallel with Code Review and Security Review.

Artifact: `07-qa-report.md`

## 8. Code Review Gate

Code Reviewer independently reviews code quality and maintainability. After implementation freeze, Code Review may run in parallel with QA and Security Review.

Artifact: `08-code-review.md`

## 9. Security Gate

Security Reviewer independently decides release security. After implementation freeze, Security Review may run in parallel with QA and Code Review, but release status must wait for all required review artifacts.

Artifact: `09-security-review.md`

## 10. Final Gate

Lead summarizes:

- what changed
- verification evidence
- QA decision
- Code Review decision
- Security decision
- remaining risks
- whether merge/release is recommended

Artifact: `10-final-summary.md`

## Fast Paths

Fast paths reduce planning overhead, not quality gates.

Fast path is allowed only when:

- scope is clear
- no new dependency is needed
- no auth, permission, privacy, payment, or deployment behavior changes
- no database migration is needed
- rollback is obvious
- the user did not ask for broader product exploration

Fast path is forbidden if any `Must Ask` trigger in `agent-team-protocol/03-ask-user-policy.md` applies.

Fast path still requires explicit user approval before execution.

### Tiny Documentation Or Text Change

```text
Lead -> Project Explorer -> User Approval -> Relevant Engineer -> QA -> Code Reviewer -> Security Reviewer -> Final
```

### Pure Backend Task

```text
Lead -> Product Gate if scope matters -> Project Explorer -> Architect -> User Approval -> Backend Engineer -> QA -> Code Reviewer -> Security Reviewer -> Final
```

### Pure Frontend Task

```text
Lead -> Product Designer -> Project Explorer -> Architect -> User Approval -> Frontend Engineer -> QA -> Code Reviewer -> Security Reviewer -> Final
```

### Pure DevOps Task

```text
Lead -> Project Explorer -> Architect if needed -> User Approval -> DevOps Engineer -> QA -> Code Reviewer -> Security Reviewer -> Final
```

Do not remove QA, Code Review, or Security Review unless the user explicitly waives them.

## Trial Workflow

For a first real test of this protocol, use:

```text
agent-team-protocol/examples/small-app-trial.md
```

This trial intentionally exercises Product Designer, Project Explorer, Architect, Frontend Engineer, QA Engineer, Code Reviewer, Security Reviewer, and Lead without requiring backend or cloud infrastructure.
