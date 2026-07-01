# Agent Team Architecture

This document explains how the Agent Team protocol works.

The user only talks to Lead. Lead routes work through gates. Specialist agents produce artifacts. Review agents inspect evidence independently. Release requires QA, Code Review, and Security Review.

## System Architecture

```mermaid
flowchart TB
    U["User"] --> L["Lead<br/>Entry point / routing / state machine"]

    L --> PD["Product Designer<br/>Goal / scope / UX"]
    L --> PE["Project Explorer<br/>Read-only codebase facts"]
    L --> RA["Research Architect<br/>Technical plan / research / architecture"]
    L --> DO1["DevOps Engineer<br/>Release feasibility review"]

    PD --> A1["Product Brief"]
    PE --> A2["Project Context Report"]
    RA --> A3["Architecture Plan"]
    DO1 --> A4["DevOps Feasibility Report"]

    A1 --> L
    A2 --> L
    A3 --> L
    A4 --> L

    L --> G["User Approval Gate<br/>Requires explicit permission"]

    G --> FE["Frontend Engineer"]
    G --> BE["Backend Engineer"]
    G --> DO2["DevOps Engineer"]

    FE --> I1["Frontend Implementation Artifact"]
    BE --> I2["Backend Implementation Artifact"]
    DO2 --> I3["DevOps Implementation Artifact"]

    I1 --> QA["QA Engineer<br/>Function / UX / tests"]
    I2 --> QA
    I3 --> QA

    QA --> CR["Code Reviewer<br/>Code quality / maintainability"]
    CR --> SR["Security Reviewer<br/>Security / release gate"]

    SR --> L
    L --> F["Final Summary<br/>Release recommendation"]
    F --> U
```

## Execution Flow

```mermaid
stateDiagram-v2
    [*] --> Intake
    Intake --> ProductGate
    ProductGate --> ContextGate
    ContextGate --> ArchitectureGate
    ArchitectureGate --> DevOpsFeasibilityGate
    DevOpsFeasibilityGate --> UserApprovalGate

    UserApprovalGate --> ImplementationGate: user approves
    UserApprovalGate --> [*]: user rejects / changes scope

    ImplementationGate --> QAGate
    QAGate --> CodeReviewGate: QA pass
    QAGate --> ImplementationGate: QA fails

    CodeReviewGate --> SecurityGate: code review pass
    CodeReviewGate --> ImplementationGate: blocking issues

    SecurityGate --> FinalGate: RELEASE_OK
    SecurityGate --> UserApprovalGate: RISK_ACCEPTANCE needed
    SecurityGate --> ArchitectureGate: RELEASE_BLOCKED / design issue
    SecurityGate --> ImplementationGate: RELEASE_BLOCKED / fix needed

    FinalGate --> [*]
```

## Core Principles

- User only commands Lead.
- Lead coordinates but does not bypass specialist judgment.
- Every agent works inside its own gate.
- Every gate produces an artifact or a clearly stated equivalent.
- Downstream agents consume artifacts, not private reasoning from upstream agents.
- Reviewers do not rely on engineer private reasoning.
- Any execution action requires explicit user permission first.
- QA, Code Review, and Security Review are hard release gates.
- Security Reviewer can veto release.

## Why This Works

```text
Subagent independent context = prevents minds from blending together
Artifact handoff = prevents workflow from blending together
Gate = prevents permission drift and skipped steps
Loop rules = allow failure recovery without restarting everything
```

## Default Gate Order

```text
Intake
 -> Product
 -> Project Context
 -> Architecture
 -> Release Feasibility
 -> User Approval
 -> Implementation
 -> QA
 -> Code Review
 -> Security
 -> Final
```

## Release Rule

Lead may summarize release readiness, but Lead cannot override QA, Code Review, or Security Review.

Security decisions are:

```text
RELEASE_OK
RELEASE_OK_WITH_RISK_ACCEPTANCE
RELEASE_BLOCKED
```

If Security returns `RELEASE_BLOCKED`, the system must not recommend release.
