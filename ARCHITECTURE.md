# Agent Team Architecture

This document explains how the Agent Team protocol works.

The user only talks to Lead. Lead routes work through gates. Specialist agents produce artifacts. Review agents inspect evidence independently. Release requires QA, Code Review, and Security Review.

Agent Team uses one Subagent Mode. Lead dispatches selected roles through bounded Role Packets and Codex custom agents in `.codex/agents/*.toml`. Lead validates each Role Packet before dispatch and validates returned artifacts before marking gates passed. Each role returns artifacts to Lead. If subagent threads are unavailable, Lead must report `SUBAGENT_UNAVAILABLE` instead of silently merging roles into one shared context.

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
        FREEZE["Implementation Freeze"]
    end

    subgraph Review["Review + Release"]
        QA["&quot;QA Engineer&quot;<br/>QA Gate"]
        CR["&quot;Code Reviewer&quot;<br/>Code Review Gate"]
        SR["&quot;Security Reviewer&quot;<br/>Security Gate"]
        MERGE["&quot;Lead&quot;<br/>Review Merge"]
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

    FE --> FREEZE
    BE --> FREEZE
    DEVI --> FREEZE

    FREEZE --> QA
    FREEZE --> CR
    FREEZE --> SR
    QA --> MERGE
    CR --> MERGE
    SR --> MERGE
    MERGE --> FINAL
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
    G --> Z["Implementation Freeze"]
    Z --> H["QA Gate<br/>&quot;QA Engineer&quot;"]
    Z --> I["Code Review<br/>&quot;Code Reviewer&quot;"]
    Z --> J["Security Gate<br/>&quot;Security Reviewer&quot;"]
    H --> M["Lead Review Merge"]
    I --> M
    J --> M
    M --> K["Final<br/>&quot;Lead&quot;"]
```

## Subagent Layer

Subagents do not change the gate order. They change how a role phase is executed.

```mermaid
flowchart LR
    U["User"] --> L["&quot;Lead&quot;<br/>parent orchestrator"]

    subgraph Subagents["Subagents"]
        PD["product_designer"]
        PE["project_explorer"]
        AR["architect"]
        FE["frontend_engineer"]
        BE["backend_engineer"]
        DO["devops_engineer"]
        QA["qa_engineer"]
        CR["code_reviewer"]
        SR["security_reviewer"]
    end

    L --> RP["Bounded Role Packet"]
    RP --> PD
    RP --> PE
    RP --> AR
    RP --> FE
    RP --> BE
    RP --> DO
    RP --> QA
    RP --> CR
    RP --> SR

    PD --> A["Artifacts"]
    PE --> A
    AR --> A
    FE --> A
    BE --> A
    DO --> A
    QA --> A
    CR --> A
    SR --> A

    A --> L
    L --> G["Gates + Loops + Final Status"]
    G --> U
```

Lead is not a subagent. Lead is the parent orchestrator.

Subagent Mode is strongest for independent exploration, architecture checks, QA, code review, and security review. Implementation subagents can write files only after explicit user approval and only when ownership boundaries are clear.

If subagent execution is unavailable, the workflow is blocked until the user enables subagents or explicitly waives subagent independence for that task.

Only Lead may dispatch subagents. After implementation freeze, Lead may fan out QA, Code Review, and Security Review in parallel. Lead then merges the returned artifacts, but does not redo or override specialist judgment.

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
- Subagent output must return as artifacts before downstream gates consume it.
- Any execution action requires explicit user permission first.
- QA, Code Review, and Security Review are hard release gates.
- Security Reviewer can veto release.

## Why This Works

```text
Subagent context = prevents minds from blending together
Role Packet = keeps each subagent bounded to allowed inputs
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
