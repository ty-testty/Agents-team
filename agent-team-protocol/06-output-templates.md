# Output Templates

## Developer Note

These templates make the protocol executable. Roles may keep outputs concise, but they should preserve these headings so downstream reviewers can parse decisions, evidence, and next steps reliably.

## General Rules

- Use explicit decisions.
- Use file references when code is involved.
- Separate facts from assumptions.
- Mark unknowns instead of guessing.
- Include evidence or say `Not run` / `Not verified`.
- Keep the artifact scoped to the role.

## Role Packet Validity Rule

A Role Packet is valid only if:

- `Role` maps to a known Codex custom agent in `.codex/agents/`.
- `Objective` is specific and scoped to the current task.
- `Approved Scope` is explicit.
- `Allowed Inputs` are listed.
- `Forbidden Inputs` are listed.
- `Evidence Included` is enough for the requested decision.
- `Evidence Missing` is explicitly marked, even if empty.
- `Required Output Artifact` matches the role.
- `Stop Conditions` are role-specific.

If a required field is missing or too vague, Lead must not dispatch the subagent. Lead must return `PACKET_INCOMPLETE`, repair the packet, and dispatch only after the packet is valid.

## Artifact Validity Rule

An artifact is valid only if:

- it includes the required decision field
- it separates evidence from assumptions
- it marks missing or unverified evidence
- it lists blockers or explicitly says none
- it includes next action or handoff
- it stays within the role scope

If an artifact lacks a required decision, evidence, or blocker status, Lead must not treat the gate as passed. Lead must mark the gate `NEEDS_MORE_EVIDENCE`, `BLOCKED`, or `NOT_RUN`, depending on the role and missing evidence.

`NEEDS_MORE_EVIDENCE` is a Lead gate status, not a replacement for role-specific artifact decisions. Role artifacts should use their own decision vocabulary and state missing evidence in the artifact body.

## Reusable Role Packet

Lead prepares this packet before dispatching each role. It is not a one-time intake artifact.

```md
# Role Packet
Role:
Objective:
Allowed Inputs:
Forbidden Inputs:
Approved Scope:
Evidence Included:
Evidence Missing:
Required Output Artifact:
Artifact Location:
Stop Conditions:
Known Unknowns:
```

## 00 Intake Summary

```md
# Intake Summary

## Request

## Task Type

## Risk Level
Low | Medium | High

## Required Gates

## Fast Path?
Yes | No

## Initial Questions

## Decision
PROCEED | NEEDS_USER_INPUT | SUBAGENT_UNAVAILABLE | PACKET_INCOMPLETE | BLOCKED
```

## 01 Product Brief

```md
# Product Brief

## User Goal

## Scope

## Non-scope

## Acceptance Criteria

## UX Flow

## UI States

## Product Risks

## Questions For User

## Decision
READY_FOR_ARCHITECTURE | NEEDS_USER_INPUT
```

## 02 Project Context Report

```md
# Project Context Report

## Relevant Files

## Existing Patterns

## Commands

## Dependencies

## Constraints

## Unknowns

## Impact Area

## Decision
CONTEXT_READY | NEEDS_MORE_EXPLORATION
```

## 03 Architecture Plan

```md
# Architecture Plan

## Project Facts

## Verified External Facts
- Claim:
- Source URL:
- Date checked:
- Confidence: High | Medium | Low
- Decision supported:

## Unverified Assumptions
- Assumption:
- Risk if wrong:
- How to verify:

## Recommendation

## Alternatives Considered

## Contracts

## Implementation Plan

## Test Strategy

## DevOps Impact

## Security Considerations

## Rollback Plan

## External Sources

## User Approval Needed
Yes | No

## Decision
READY_FOR_APPROVAL | NEEDS_USER_INPUT | BLOCKED
```

## 04 DevOps Feasibility

```md
# DevOps Feasibility Report

## Runtime Impact

## Env And Secrets

## CI/CD Impact

## Migration Or Release Plan

## Rollback

## Observability

## Blocking Deployment Risks

## Evidence

## Decision
FEASIBLE | FEASIBLE_WITH_NOTES | BLOCKED
```

## 05 Approval Request

```md
# Approval Request

## Proposed Scope

## Proposed Technical Plan

## Release Impact

## Risks

## What Approval Allows

## What Is Still Not Allowed

## Decision Needed
APPROVE_IMPLEMENTATION | REQUEST_CHANGES | CANCEL
```

## 06 Engineering Implementation

```md
# <Frontend | Backend | DevOps> Implementation

## Decision
IMPLEMENTED | NOT_APPLICABLE | BLOCKED

## Scope Handled

## Files Changed

## Implementation Slices
- Slice Goal:
- Files Touched:
- Behavior Changed:
- Checks Run:
- Rollback Note:
- Next Slice:

## Behavior Changed

## Self-check

## Evidence

## Handoffs

## Implementation Freeze
Frozen Diff Or Artifact:
Review Fan-out Ready: Yes | No

## Risks
```

## 07 QA Report

```md
# QA Report

## Decision
PASS | FAIL | NOT_RUN

## Commands Run

## Evidence

## Results

## Acceptance Criteria Check

## UX/UI Check

## Accessibility Check

## Browser And Interaction Check

## Not Verified

## Blockers

## Non-blocking Issues

## Next Action
```

## 08 Code Review

```md
# Code Review Report

## Decision
PASS | PASS_WITH_NOTES | BLOCKED

## Findings

## Evidence

## Not Verified

## Blockers

## Non-blocking Issues

## Missing Tests

## Doubt Check
Claim:
Evidence:
What could make this wrong?
Cheapest verification:
Decision:

## Maintainability Notes

## Next Action
```

## 09 Security Review

```md
# Security Review

## Security Decision
RELEASE_OK | RELEASE_OK_WITH_RISK_ACCEPTANCE | RELEASE_BLOCKED

## Reviewed Areas

Use `Reviewed`, `Not relevant`, or `Not checked` for important areas.

## Findings

## Evidence

## Not Verified

## Blockers

## Risk Acceptance Items

## Next Action

## Release Recommendation
```

## 10 Final Summary

```md
# Final Summary

## Outcome

## Files Changed

## Verification

## QA Decision

## Code Review Decision

## Security Decision

## Review Merge
QA Artifact:
Code Review Artifact:
Security Artifact:
Stale Review Artifacts:
Conflicts:

## Remaining Risks

## Release Recommendation
READY_TO_RELEASE | NOT_READY_TO_RELEASE | NEEDS_USER_RISK_ACCEPTANCE

If QA is `NOT_RUN`, use `NOT_READY_TO_RELEASE` unless the user explicitly waives the verification gap; even then, state the waiver instead of calling the result ready.
```
