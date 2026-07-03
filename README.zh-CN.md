# Agent Team Protocol

一个可移植的 Markdown 协议，用来让 Codex 在任何代码仓库中像一个带 gate 的 AI 软件开发团队一样工作。

## Developer Note

这个项目现在刻意不包含自定义 runtime。runtime 就是 Codex 本身。`AGENTS.md` 告诉 Codex 要读取哪些内容，`agent-team-protocol/` 保存稳定的团队规则。

维护这个仓库时，请保持这个区别清楚：

- `AGENTS.md` 是 Codex 首先看到的小型 loader。
- `agent-team-protocol/` 是 Codex 随后读取的详细协议。
- `INSTALL.md` 说明如何把协议复制到另一个仓库。

对于 Codex 风格的使用方式，本仓库包含 [AGENTS.md](AGENTS.md)，这是一个 AI 可读取的 loader 文件。详细规则位于 `agent-team-protocol/`。把 `AGENTS.md` 和 `agent-team-protocol/` 一起复制到任何目标仓库根目录后，你就可以正常和 Codex 对话，同时让 Codex 遵循相同的团队角色、gate、artifact handoff、loop rules 和 release checks。

复制/安装说明见 [INSTALL.md](INSTALL.md)。

这个设计遵循本次对话中定义的角色和 gate：

- Lead
- Product Designer
- Project Explorer
- Architect
- Frontend Engineer
- Backend Engineer
- DevOps Engineer
- QA Engineer
- Code Reviewer
- Security Reviewer

最重要的思想是 artifact handoff。Agents 不自由聊天。每个 gate 产出结构化 artifact，下一个 gate 只读取自己需要的 artifact。

## Engineering Model

这个协议使用 sub-agent orchestration model，并配合 workflow gates。

实际含义是：

- 用户只和 Lead 对话。
- Lead 作为 orchestrator。
- 每个 role 都像一个专门的 sub-agent。
- 每个 specialist 只接收自己被允许使用的输入。
- 每个 specialist 输出 artifact，而不是加入自由形式的群聊。
- Review roles 独立判断 evidence，而不是相信 implementation reasoning。

当真实 multi-agent runtime 可用时，每个 role 应该运行在自己的 context window 中。

当只有一个 Codex conversation 可用时，本协议通过 `agent-team-protocol/09-context-boundaries.md` 模拟 sub-agent 隔离。Lead 为每个 role 打包 allowed inputs，每个 role 必须忽略 forbidden inputs，例如另一个 role 的 private reasoning、无关 transcript history，或未批准的 scope changes。

团队也使用 Agent Team 风格的协作，但只通过受控的 artifacts、gates 和 loops：

- artifacts 保存 decisions 和 evidence
- gates 防止步骤被跳过
- loops 把失败的工作送回最近的 responsible role
- QA、Code Review 和 Security 保持独立 release checks

最终结果刻意不是自由聊天式 agent swarm。它更接近：

```text
Lead orchestrator
 -> isolated specialist sub-agents
 -> artifact handoff
 -> gated review
 -> repair loops
 -> final release decision
```

## Quality Rules

这个协议保持额外严格性是轻量的。它不会要求每个小任务都走完整仪式，但会在错误代价较高的地方加强检查：

- Lead 为每个 role 打包 allowed inputs、evidence included、evidence missing 和 expected output。
- Architect 区分 repository facts、verified external facts、assumptions 和 recommendations。
- Architect 为 verified external facts 记录 source URL、date checked、confidence 和 supported decision。
- Engineers 以小的 implementation slices 工作，并记录 checks 和 rollback notes。
- 当涉及 UI 且工具可用时，QA 检查 browser behavior、console errors 和 core interactions。
- Code Reviewer 对最有风险的 correctness claim 做一个简短 doubt check。
- Security Reviewer 把重要区域标记为 `Reviewed`、`Not relevant` 或 `Not checked`。
- 只要触发 must-ask 条件，就禁止 fast path。
- 同类 blocking failure 连续 2 次未解决后停止 loop。

这些规则的目的，是在不把每个任务变成重流程的前提下，提高独立性和 evidence quality。

## Quick Start

把这些复制到任何你希望 Codex 遵循该团队协议的项目根目录：

```text
AGENTS.md
agent-team-protocol/
```

然后从该项目根目录开始一个新的 Codex session，并正常对话：

```text
Help me add login.
Review whether this branch is ready to release.
Refactor the dashboard export flow.
```

Codex 应该把你当作在和 Lead 对话，然后通过 protocol gates 路由工作。

## Workflow

```text
Lead
 -> Product Designer
 -> Project Explorer
 -> Architect
 -> DevOps Engineer, early release feasibility
 -> User Approval
 -> Frontend Engineer / Backend Engineer / DevOps Engineer
 -> QA Engineer
 -> Code Reviewer
 -> Security Reviewer
 -> Lead Final
```

Implementation 在 user approval 之前被阻塞。

Security Reviewer 是 hard release gate，并且可以 veto release。

Implementation 阶段，本协议期望分开的 engineering handoffs：

```text
06a-frontend-implementation.md
06b-backend-implementation.md
06c-devops-implementation.md
06-implementation-summary.md
```

QA、Code Reviewer 和 Security Reviewer 会独立读取这些 artifacts。

## File Map

```text
AGENTS.md
  Loader file. 保持简短。它告诉 Codex 要读取哪些 protocol files。

agent-team-protocol/
  gates、artifacts、approval、loops 和 release policy 的主要规则。

agent-team-protocol/roles/
  每个 role 一个文件。把 role-specific rules 放在这里，而不是放进 AGENTS.md。

agent-team-protocol/examples/
  在真实仓库中测试该 protocol 的 trial workflows。

agent-team-protocol/artifacts/
  可选的临时任务 artifacts。默认被 git 忽略，除非用户明确要求保留。

INSTALL.md
  复制/安装到另一个仓库的说明。
```

## Ask-User Policy

协议应该在以下情况继续前询问用户：

- Scope 或 acceptance criteria 不清楚。
- 需要 product tradeoff。
- 引入新的 dependency、paid service、external account 或 vendor lock-in。
- Database schema、migrations、auth、permissions 或 privacy behavior 发生变化。
- Deployment、CI/CD、environment variables、secrets、cron、workers 或 cloud resources 发生变化。
- 删除功能、重写模块，或需要 broad refactor。
- Reviewers 对 release readiness 有分歧。
- Security 返回 `RELEASE_OK_WITH_RISK_ACCEPTANCE` 或 `RELEASE_BLOCKED`。
- 任务明显超出原始 scope 或 budget。

如果触发以上任一条件，就不允许 fast path。

## Loop Rules

```text
QA failed
 -> responsible engineer
 -> QA rerun
 -> Code Review and Security when relevant

Code Review blocked
 -> responsible engineer
 -> focused QA
 -> Code Reviewer rereview

Security blocked
 -> Architect or responsible engineer
 -> QA
 -> Code Review
 -> Security rereview

Scope changed
 -> Product Gate
 -> regenerate downstream artifacts
 -> user approval

同类 blocking failure 连续 2 次未解决
 -> stop
 -> Lead 汇报失败内容、已尝试内容、剩余选项和推荐下一步
 -> 第 3 次尝试需要用户明确批准
```

## Design References

本协议受到 OpenAI、Anthropic 和 agent-skills 公开 agent architecture / engineering workflow patterns 的影响：

- [OpenAI Codex Subagents](https://developers.openai.com/codex/concepts/subagents)
- [OpenAI Agents SDK Handoffs](https://openai.github.io/openai-agents-python/handoffs/)
- [Anthropic Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)
- [agent-skills: Orchestration Patterns](https://github.com/addyosmani/agent-skills/blob/main/references/orchestration-patterns.md)

本协议把这些思想组合成一个可移植到任何仓库的 Markdown system：sub-agent style role isolation、artifact handoff、explicit gates、user approval、review independence、security veto 和 recovery loops。
