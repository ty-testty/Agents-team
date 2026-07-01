# Agent Team Architecture

This document explains how the Agent Team protocol works.

The user only talks to Lead. Lead routes work through gates. Specialist agents produce artifacts. Review agents inspect evidence independently. Release requires QA, Code Review, and Security Review.

## System Architecture

This diagram is intentionally square and layered. Solid arrows are the normal path. Dotted arrows are recovery paths.

```mermaid
flowchart LR
    subgraph Entry["Entry"]
        U["User"]
        L["&quot;Lead&quot;<br/>Entry + routing"]
    end

    subgraph Planning["Planning Gates"]
        PD["&quot;Product Designer&quot;<br/>Product Gate"]
        PE["&quot;Project Explorer&quot;<br/>Context Gate"]
        RA["&quot;Architect&quot;<br/>Architecture Gate"]
        DEVF["&quot;DevOps Engineer&quot;<br/>Feasibility Gate"]
    end

    subgraph Approval["Approval"]
        APPR["User Approval Gate<br/>explicit permission required"]
    end

    subgraph Implementation["Implementation"]
        FE["&quot;Frontend Engineer&quot;"]
        BE["&quot;Backend Engineer&quot;"]
        DEVI["&quot;DevOps Engineer&quot;"]
    end

    subgraph Review["Review + Release"]
        QA["&quot;QA Engineer&quot;<br/>QA Gate"]
        CR["&quot;Code Reviewer&quot;<br/>Code Review Gate"]
        SR["&quot;Security Reviewer&quot;<br/>Security Gate"]
        FINAL["&quot;Lead&quot;<br/>Final Summary"]
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
    A["Intake<br/>&quot;Lead&quot;"] --> B["Product Gate<br/>&quot;Product Designer&quot;"]
    B --> C["Context Gate<br/>&quot;Project Explorer&quot;"]
    C --> D["Architecture Gate<br/>&quot;Architect&quot;"]
    D --> E["Release Feasibility<br/>&quot;DevOps Engineer&quot;"]
    E --> F["User Approval<br/>&quot;Lead&quot;"]
    F --> G["Implementation<br/>&quot;Frontend Engineer&quot; / &quot;Backend Engineer&quot; / &quot;DevOps Engineer&quot;"]
    G --> H["QA Gate<br/>&quot;QA Engineer&quot;"]
    H --> I["Code Review<br/>&quot;Code Reviewer&quot;"]
    I --> J["Security Gate<br/>&quot;Security Reviewer&quot;"]
    J --> K["Final<br/>&quot;Lead&quot;"]
```

## Rejection And Recovery Loops

User rejection is not always the end. Lead classifies the reason, sends the work back to the nearest responsible gate, then returns the corrected work to the matching approval or review gate.

```mermaid
flowchart LR
    subgraph UserLoop["User Approval Loop"]
        UFAIL["User rejects<br/>or changes scope"] --> ULEAD["&quot;Lead&quot;<br/>classifies reason"]
        ULEAD --> UCANCEL["Cancel task<br/>End"]
        ULEAD --> UOWNER["Responsible Gate<br/>Product / Architecture / DevOps / Security"]
        UOWNER --> UREWORK["Revise artifact"]
        UREWORK --> UAPPROVAL["User Approval Gate"]
        UAPPROVAL -. rejects again .-> UFAIL
    end

    subgraph QALoop["QA Loop"]
        QAFAIL["QA fails"] --> QAFIX["Implementation Gate<br/>Frontend / Backend / DevOps"]
        QAFIX --> QARERUN["Rerun QA Gate"]
        QARERUN -. still fails .-> QAFAIL
    end

    subgraph ReviewLoop["Code Review Loop"]
        CRBLOCK["Code Review blocks"] --> CRLEAD["&quot;Lead&quot;<br/>classifies issue"]
        CRLEAD --> CRFIX["Implementation Gate"]
        CRLEAD --> CRARCH["Architecture Gate"]
        CRFIX --> CRRERUN["Rerun Code Review"]
        CRARCH --> CRRERUN
        CRRERUN -. still blocked .-> CRBLOCK
    end

    subgraph SecurityLoop["Security Loop"]
        SECBLOCK["Security blocks"] --> SECLEAD["&quot;Lead&quot;<br/>classifies issue"]
        SECLEAD --> SECFIX["Implementation Gate"]
        SECLEAD --> SECARCH["Architecture Gate"]
        SECLEAD --> SECRISK["Risk Acceptance<br/>User Approval Gate"]
        SECFIX --> SECRERUN["Rerun Security Gate"]
        SECARCH --> SECRERUN
        SECRISK --> SECRERUN
        SECRERUN -. still blocked .-> SECBLOCK
    end
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
