# Review Checklists

## Developer Note

Checklists keep review roles from becoming vague. Use them as prompts for judgment, not as mechanical boxes to tick blindly.

## QA Checklist

Functional:

- Acceptance criteria are satisfied.
- Main user path works.
- Important edge cases are covered.
- Error behavior is visible and understandable.
- Loading and empty states behave correctly.
- Regression-sensitive paths were considered.

Commands:

- Tests run, or reason not run is stated.
- Lint run when available, or reason not run is stated.
- Typecheck run when available, or reason not run is stated.
- Build run when available, or reason not run is stated.

UX/UI:

- Layout works on expected desktop size.
- Layout works on expected mobile size when UI is involved.
- Text does not overflow.
- Buttons and controls have clear states.
- Forms have validation and useful errors.
- Focus, keyboard, labels, and contrast are acceptable at a basic level.

Evidence:

- Include command output summary.
- Include screenshots or manual steps for UI when possible.
- Mark anything unverified.

## Code Review Checklist

Correctness:

- The diff matches the approved scope.
- Edge cases are handled.
- Errors are handled close to the boundary.
- Data transformations are explicit and tested when meaningful.

Maintainability:

- Code follows local patterns.
- Names are clear.
- No needless abstraction was introduced.
- Duplication is acceptable or reduced.
- Public behavior changes are documented when needed.

Tests:

- Tests cover the main behavior.
- Tests cover important failure cases.
- Snapshot or brittle tests are avoided unless appropriate.
- Missing tests are called out.

Integration:

- Frontend and backend contracts match.
- Migration and deployment assumptions are clear.
- Configuration changes are documented.

## Security Checklist

Authentication and authorization:

- New paths enforce auth when required.
- Authorization is checked server-side.
- Role/tenant/user boundaries are preserved.

Input and output:

- Inputs are validated.
- User-controlled content is escaped or sanitized where needed.
- Error messages do not leak sensitive internals.
- Logs do not expose secrets or personal data.

Data:

- Queries cannot expose another user's data.
- Migrations avoid destructive surprises.
- Sensitive data is encrypted or protected according to project norms.

Browser and network:

- XSS risks are considered.
- CSRF risks are considered for state-changing browser requests.
- SSRF or open redirect risks are considered when URLs are accepted.
- CORS, cookies, and sessions are not weakened.

Dependencies and config:

- New dependencies are justified and approved.
- Secrets are not committed.
- Env vars are documented without values.
- Deployment exposure does not widen accidentally.

Decision:

- Use `RELEASE_BLOCKED` for exploitable or high-impact unresolved risks.
- Use `RELEASE_OK_WITH_RISK_ACCEPTANCE` only when the risk is real but the user may choose to accept it.
- Use `RELEASE_OK` when no blocking or acceptance-needed security risk remains.
