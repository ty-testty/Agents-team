# Ask-User Policy

## Developer Note

This file controls interruption. The protocol should ask the user for ownership decisions, not for every implementation detail. When in doubt, ask only if the decision changes product scope, risk, cost, data, security, or release behavior.

Ask the user when a decision belongs to the product owner, risk owner, or release owner.

Do not ask the user for every minor implementation detail. Make reasonable local decisions when they fit the approved plan and project conventions.

## Execution Permission

Always ask the user before executing implementation work.

This includes:

- creating files
- modifying files
- deleting files
- moving or renaming files
- running commands that change project state
- installing dependencies
- launching dev servers
- downloading tools or browsers
- applying patches
- generating app/game/tool artifacts

The user's task request is not enough. Wait for a separate clear approval such as:

- "可以执行"
- "approve"
- "go ahead"
- "开始做"
- "yes, implement it"

## Must Ask

Ask the user before continuing when:

- scope or acceptance criteria are unclear
- product tradeoffs are required
- a new dependency is introduced
- a paid service, external account, or vendor lock-in is introduced
- database schema or migrations change
- auth, permissions, or privacy behavior changes
- deployment, CI/CD, env vars, secrets, cron, workers, queues, storage, or cloud resources change
- a feature is deleted
- a module is rewritten
- broad refactoring is required
- QA, Code Reviewer, and Security Reviewer disagree on release readiness
- Security returns `RELEASE_OK_WITH_RISK_ACCEPTANCE`
- Security returns `RELEASE_BLOCKED`
- task cost or scope expands materially beyond the original request

## Usually Do Not Ask

After execution has been explicitly approved, do not stop to ask for every local implementation detail. Do not stop to ask for:

- following existing project style
- adding focused tests for changed behavior
- fixing lint/typecheck/build errors caused by the change
- local variable/function naming when conventional
- minor responsive or accessibility fixes within approved UI intent
- non-user-visible refactors needed to keep the change clean and small

## Ask Shape

When asking, be concrete:

- state the decision needed
- list the tradeoff
- recommend a default when appropriate
- explain what will happen next

Avoid vague questions like "What should I do?"
