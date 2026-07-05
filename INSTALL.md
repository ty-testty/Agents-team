# Install Agent Team In Another Repository

## Developer Note

The installable unit is `AGENTS.md`, `agent-team-protocol/`, and `.codex/`. Do not copy only `AGENTS.md`; it is just a loader and will intentionally stop Codex if the protocol directory is missing.

Copy these into the target repository root:

```text
AGENTS.md
agent-team-protocol/
.codex/
```

`AGENTS.md` is the Codex loader. `agent-team-protocol/` contains the detailed rules. `.codex/` contains Codex custom agent definitions for Subagent Mode.

Task artifacts, when written as files, should go under:

```text
agent-team-protocol/artifacts/<task-id>/
```

That directory is temporary by default and should be ignored unless the user explicitly asks to preserve artifacts.

## Install

From this repository:

```bash
cp AGENTS.md /path/to/target-repo/AGENTS.md
cp -R agent-team-protocol /path/to/target-repo/agent-team-protocol
cp -R .codex /path/to/target-repo/.codex
```

Then start a new Codex session from the target repository root.

## Verify

Ask Codex:

```text
Summarize the active Agent Team protocol.
```

It should mention:

- Lead as the user entry point
- gated workflow
- artifact handoff
- Subagent Mode
- Codex custom agents under `.codex/agents/`
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
