# Loop Rules

## Developer Note

Loops are the recovery system. Do not restart the whole workflow for every failure. Return to the nearest responsible gate, fix the cause, then rerun only the necessary downstream gates.

Most software work does not succeed in one pass. Return to the nearest responsible gate, not the beginning of the whole process.

## QA Fails

```text
QA failed
 -> responsible engineer
 -> focused fix
 -> QA rerun
 -> Code Review when diff changed
 -> Security Review when relevant
```

## Code Review Blocks

```text
Code Review blocking issue
 -> responsible engineer
 -> focused fix
 -> QA focused rerun
 -> Code Reviewer rereview
 -> Security Review when relevant
```

## Security Blocks

```text
Security RELEASE_BLOCKED
 -> Architect if design must change, otherwise responsible engineer
 -> fix
 -> QA
 -> Code Review
 -> Security rereview
```

## Architecture Is Wrong Or Unclear

```text
Architecture issue
 -> Product Designer, Project Explorer, or Architect
 -> revised plan
 -> user approval if scope or risk changed
```

## Scope Changes

```text
Scope changed
 -> Product Gate
 -> regenerate downstream artifacts
 -> user approval
```

## DevOps Feasibility Fails

```text
Release feasibility issue
 -> Architect if plan must change
 -> DevOps Engineer if runtime plan must change
 -> user approval if release impact changed
```

## Product Or UX Problem Found Late

```text
Late product/UX issue
 -> Product Designer
 -> Architect if technical shape changes
 -> responsible engineer
 -> QA
 -> Code Review
 -> Security Review when relevant
```

## Loop Limit

After repeated failure of the same kind, stop and ask the user with a concise summary:

- what failed
- what was tried
- what options remain
- recommended next step
