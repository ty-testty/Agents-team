# Release Policy

## Developer Note

This file is the final safety boundary. Lead can summarize, but QA, Code Review, and Security own their gates. Security can veto release.

Release readiness is decided by evidence, not optimism.

## Hard Rules

- No release recommendation without QA evidence.
- No release recommendation without Code Review.
- No release recommendation without Security Review.
- Security `RELEASE_BLOCKED` blocks release.
- `RELEASE_OK_WITH_RISK_ACCEPTANCE` requires explicit user acceptance.
- If tests cannot be run, say so in the final summary.
- If external facts were needed but could not be verified, say so.
- If QA is `NOT_RUN`, release must be `NOT_READY_TO_RELEASE`; the user may explicitly waive the verification gap, but the protocol must not describe the result as ready.
- If Code Review is `BLOCKED`, release must not be recommended.

## Required Final Summary

Lead final response should include:

- files changed
- behavior changed
- verification run
- QA decision
- Code Review decision
- Security decision
- remaining risks
- whether merge/release is recommended

## Security Decisions

Security Reviewer must return exactly one of:

- `RELEASE_OK`
- `RELEASE_OK_WITH_RISK_ACCEPTANCE`
- `RELEASE_BLOCKED`

## Risk Acceptance

Only the user can accept release risk. Lead, engineers, QA, and Code Reviewer cannot accept security risk on behalf of the user.

## Evidence Standards

Good evidence:

- command output
- passing tests
- build/lint/typecheck results
- screenshots for UI
- reproduction steps
- diff review with file references
- security reasoning tied to code/config
- explicit not-applicable reasoning for skipped roles

Weak evidence:

- "looks good"
- "should work"
- "probably safe"
- assumptions without commands, code references, or screenshots
