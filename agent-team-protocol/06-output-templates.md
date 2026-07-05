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

## Reusable Role Packet

Lead prepares this packet before dispatching each role. It is not a one-time intake artifact.

```md
# Role Packet
Role:
Objective:
Dispatch Mode:
SIMULATED | NATIVE_SUBAGENT
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
PROCEED | NEEDS_USER_INPUT | BLOCKED
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

## Handoffs

## Risks
```

## 07 QA Report

```md
# QA Report

## Decision
PASS | FAIL | NOT_RUN

## Commands Run

## Results

## Acceptance Criteria Check

## UX/UI Check

## Accessibility Check

## Browser And Interaction Check

## Evidence Missing Or Not Run

## Blocking Issues

## Non-blocking Issues

## Recommended Rerun
```

## 08 Code Review

```md
# Code Review Report

## Decision
PASS | PASS_WITH_NOTES | BLOCKED

## Findings

## Blocking Issues

## Non-blocking Issues

## Missing Tests

## Doubt Check
Claim:
Evidence:
What could make this wrong?
Cheapest verification:
Decision:

## Maintainability Notes

## Required Fixes
```

## 09 Security Review

```md
# Security Review

## Security Decision
RELEASE_OK | RELEASE_OK_WITH_RISK_ACCEPTANCE | RELEASE_BLOCKED

## Reviewed Areas

Use `Reviewed`, `Not relevant`, or `Not checked` for important areas.

## Findings

## Blocking Issues

## Risk Acceptance Items

## Required Fixes

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

## Remaining Risks

## Release Recommendation
READY_TO_RELEASE | NOT_READY_TO_RELEASE | NEEDS_USER_RISK_ACCEPTANCE

If QA is `NOT_RUN`, use `NOT_READY_TO_RELEASE` unless the user explicitly waives the verification gap; even then, state the waiver instead of calling the result ready.
```
