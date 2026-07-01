# Agent Team Architecture

This document explains how the Agent Team protocol works.

The user only talks to Lead. Lead routes work through gates. Specialist agents produce artifacts. Review agents inspect evidence independently. Release requires QA, Code Review, and Security Review.

## System Architecture

This diagram is intentionally square and layered. Solid arrows are the normal path. Dotted arrows are recovery paths.

```mermaid
flowchart LR
    subgraph Entry["Entry"]
        U["User"]
        L["Lead<br/>Entry + routing"]
    end

    subgraph Planning["Planning Gates"]
        PD["Product Designer<br/>Product Gate"]
        PE["Project Explorer<br/>Context Gate"]
        RA["Research Architect<br/>Architecture Gate"]
        DEVF["DevOps Engineer<br/>Feasibility Gate"]
    end

    subgraph Approval["Approval"]
        APPR["User Approval Gate<br/>explicit permission required"]
    end

    subgraph Implementation["Implementation"]
        FE["Frontend Engineer"]
        BE["Backend Engineer"]
        DEVI["DevOps Engineer"]
    end

    subgraph Review["Review + Release"]
        QA["QA Engineer<br/>QA Gate"]
        CR["Code Reviewer<br/>Code Review Gate"]
        SR["Security Reviewer<br/>Security Gate"]
        FINAL["Lead<br/>Final Summary"]
    end

    U --> L
    L --> PD
    L --> PE
    L --> RA
    L --> DEVF

    PD --> APPR
    PE --> APPR
    RA --> APPR
    DEVF --> APPR

    APPR --> FE
    APPR --> BE
    APPR --> DEVI

    FE --> QA
    BE --> QA
    DEVI --> QA

    QA --> CR
    CR --> SR
    SR --> FINAL
    FINAL --> U

    APPR -. scope change .-> PD
    APPR -. technical concern .-> RA
    APPR -. release concern .-> DEVF
    QA -. fails .-> FE
    QA -. fails .-> BE
    QA -. fails .-> DEVI
    CR -. architecture issue .-> RA
    SR -. risk acceptance .-> APPR
    SR -. design issue .-> RA
```

## Execution Flow

This diagram shows only the main gate order. Rejections and failures are handled in the loop diagram below.

```mermaid
flowchart LR
    A["Intake<br/>Lead"] --> B["Product Gate<br/>Product Designer"]
    B --> C["Context Gate<br/>Project Explorer"]
    C --> D["Architecture Gate<br/>Research Architect"]
    D --> E["Release Feasibility<br/>DevOps Engineer"]
    E --> F["User Approval<br/>Lead"]
    F --> G["Implementation<br/>Frontend / Backend / DevOps"]
    G --> H["QA Gate<br/>QA Engineer"]
    H --> I["Code Review<br/>Code Reviewer"]
    I --> J["Security Gate<br/>Security Reviewer"]
    J --> K["Final<br/>Lead"]
```

## Rejection And Recovery Loops

User rejection is not always the end. Lead classifies the reason and sends the work back to the nearest responsible gate.

```mermaid
flowchart TB
    REJ["User rejects approval"] --> LEAD["Lead classifies reason"]

    LEAD --> CANCEL["Cancel task<br/>End"]
    LEAD --> SCOPE["Scope / product change<br/>Product Gate"]
    LEAD --> TECH["Technical concern<br/>Architecture Gate"]
    LEAD --> REL["Release / cost concern<br/>DevOps Feasibility"]
    LEAD --> RISK["Risk concern<br/>Security or Architecture"]

    QAFAIL["QA fails"] --> IMPL["Implementation Gate"]

    CRBLOCK["Code Review blocks"] --> CRWHY["Lead classifies issue"]
    CRWHY --> IMPLISSUE["Implementation issue<br/>Implementation Gate"]
    CRWHY --> ARCHISSUE["Architecture issue<br/>Architecture Gate"]

    SECBLOCK["Security blocks"] --> SECWHY["Lead classifies issue"]
    SECWHY --> FIX["Fix needed<br/>Implementation Gate"]
    SECWHY --> DESIGN["Design issue<br/>Architecture Gate"]
    SECWHY --> ACCEPT["Risk acceptance needed<br/>User Approval Gate"]
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
