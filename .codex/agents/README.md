# Codex Agent Mapping

This directory contains Codex custom agent definitions for Agent Team Subagent Mode.

Codex identifies custom agents by the TOML `name` field, not by filename. Keep the TOML filename, `name`, and role spec aligned so Lead can dispatch the correct subagent with a bounded Role Packet.

| Codex TOML | Agent name | Role spec |
|---|---|---|
| `architect.toml` | `architect` | `agent-team-protocol/roles/architect.md` |
| `backend-engineer.toml` | `backend_engineer` | `agent-team-protocol/roles/backend-engineer.md` |
| `code-reviewer.toml` | `code_reviewer` | `agent-team-protocol/roles/code-reviewer.md` |
| `devops-engineer.toml` | `devops_engineer` | `agent-team-protocol/roles/devops-engineer.md` |
| `frontend-engineer.toml` | `frontend_engineer` | `agent-team-protocol/roles/frontend-engineer.md` |
| `product-designer.toml` | `product_designer` | `agent-team-protocol/roles/product-designer.md` |
| `project-explorer.toml` | `project_explorer` | `agent-team-protocol/roles/project-explorer.md` |
| `qa-engineer.toml` | `qa_engineer` | `agent-team-protocol/roles/qa-engineer.md` |
| `security-reviewer.toml` | `security_reviewer` | `agent-team-protocol/roles/security-reviewer.md` |

## Maintenance Rule

When adding, renaming, or removing a role, update all three surfaces together:

- `.codex/agents/*.toml`
- `agent-team-protocol/roles/*.md`
- `agent-team-protocol/10-subagents.md`

If any surface is missing or mismatched, Lead must treat dispatch as blocked until the mapping is repaired.
