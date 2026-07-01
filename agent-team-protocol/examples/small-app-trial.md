# Small App Trial

## Developer Note

Use this example to test whether the protocol feels good in a real Codex session. The point is not to build a large product; the point is to watch the gates, handoffs, and reviews work on a small app.

## Recommended Trial App

Build a tiny task board app:

- Add tasks.
- Mark tasks done.
- Filter all / active / done.
- Persist locally.
- Show empty state.
- Work on mobile and desktop.

This is small enough to finish quickly but rich enough to exercise Product Designer, Frontend Engineer, QA Engineer, Code Reviewer, and Security Reviewer.

## Suggested User Prompt

```text
Using the Agent Team protocol, build a tiny task board app in this repository. Keep it local-only, no backend, no auth, no external services. Ask me before adding dependencies.
```

## Expected Fast Path

```text
Lead
 -> Product Designer
 -> Project Explorer
 -> Research Architect
 -> User Approval
 -> Frontend Engineer
 -> QA Engineer
 -> Code Reviewer
 -> Security Reviewer
 -> Lead Final
```

DevOps and Backend should be `NOT_APPLICABLE` unless the chosen stack requires build or setup changes.

## What To Watch

Good behavior:

- Lead asks for approval before implementation.
- Product Designer defines empty, done, and filter states.
- Project Explorer checks existing files before coding.
- Research Architect keeps the solution simple.
- Frontend Engineer avoids unnecessary dependencies.
- QA checks mobile/desktop and empty states.
- Code Reviewer checks local persistence and state edge cases.
- Security Reviewer notes that local-only storage has limited release risk but should not store sensitive data.

Bad behavior:

- Engineering starts before approval.
- The agent adds a backend without need.
- The agent adds a UI library without asking.
- QA says "looks good" without running or inspecting anything.
- Security is skipped because the app is small.

## Acceptance Criteria

- User can add a task.
- User can mark a task done and undone.
- User can delete a task.
- User can filter all / active / done.
- Tasks persist after refresh.
- Empty state is clear.
- Layout works on mobile and desktop.
- No external service is used.
- No production dependency is added without approval.
