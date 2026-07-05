# Install Agent Team In Another Repository

## Developer Note

The required installable unit is `AGENTS.md` plus `agent-team-protocol/`. Do not copy only `AGENTS.md`; it is just a loader and will intentionally stop Codex if the protocol directory is missing.

The native Codex subagent adapter is `.codex/`. Copy it when you want project-scoped Codex custom agents in the target repository.

Copy these into the target repository root:

```text
AGENTS.md
agent-team-protocol/
```

`AGENTS.md` is the Codex loader. `agent-team-protocol/` contains the detailed rules.

Task artifacts, when written as files, should go under:

```text
agent-team-protocol/artifacts/<task-id>/
```

That directory is temporary by default and should be ignored unless the user explicitly asks to preserve artifacts.

## Minimal Install

From this repository:

```bash
cp AGENTS.md /path/to/target-repo/AGENTS.md
cp -R agent-team-protocol /path/to/target-repo/agent-team-protocol
```

Then start a new Codex session from the target repository root.

## Native Subagent Install

To add project-scoped Codex custom agents, also copy:

```bash
cp -R .codex /path/to/target-repo/.codex
```

Native subagents should be used only when explicitly requested by the user or enabled by a standing user instruction for that repository.

## Verify

Ask Codex:

```text
Summarize the active Agent Team protocol.
```

It should mention:

- Lead as the user entry point
- gated workflow
- artifact handoff
- simulated mode by default
- native Codex subagents when explicitly requested or enabled by a standing user instruction
- user approval before implementation
- QA, Code Review, and Security release gates
- Security Reviewer veto power

## Trial

After installing, try the small app workflow in:

```text
agent-team-protocol/examples/small-app-trial.md
```

## Customization

For a specialized subdirectory, add a closer:

```text
AGENTS.override.md
```

or another nested:

```text
AGENTS.md
```

Use nested files only for local overrides. Keep the shared protocol in `agent-team-protocol/`.
