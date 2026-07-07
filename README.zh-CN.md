# Agent Team

Agent Team 是一套可移植的 Markdown 协议，用来让 Codex 在任何代码仓库里更像一个有纪律的软件开发团队。

你仍然像平常一样和 Codex 对话。区别是，Codex 会按照团队流程工作：先规划，执行前先询问，改动必须在批准范围内，然后经过 QA、Code Review 和 Security Review，最后才建议是否可以 release。

## What This Is

这个仓库给 Codex 一份可复用的团队工作手册。

用更直白的话说：

1. 你只和 `Lead` 对话。
2. `Lead` 给专业 subagents 创建有边界的任务包。
3. 专业 subagents 返回结构化 artifacts。
4. QA、Code Review 和 Security 独立检查 evidence。
5. `Lead` 汇总结果，并告诉你是否建议 release。

它刻意不是自由聊天式 agent swarm。Roles 之间不会随意聊天；它们通过 artifacts 交接工作。

```text
User
 -> Lead
 -> specialist subagents
 -> artifacts
 -> gates and recovery loops
 -> final release decision
```

## Key Terms

- `Lead`：你唯一直接对话的角色。Lead 负责任务路由、请求批准、验证 artifacts，并汇总最终状态。
- `Subagent`：运行在独立上下文里的专业角色，比如 Architect、QA Engineer 或 Security Reviewer。
- `Gate`：流程里的检查点。每个 gate 必须产出 artifact 或明确 decision，工作才能继续。
- `Artifact`：结构化交接结果。它记录 decisions、evidence、missing evidence、blockers 和 next action。
- `Role Packet`：Lead 发给 subagent 的有边界任务包。里面写明 objective、allowed inputs、forbidden inputs、evidence 和 expected output。
- `Implementation Freeze`：实现完成后的冻结点。之后 QA、Code Review 和 Security 基于同一份 diff 或 implementation artifact 检查。
- `Security Veto`：Security Reviewer 可以用 `RELEASE_BLOCKED` 阻止 release；Lead 不能覆盖它。
- `Runtime`：真正自动运行 agents 的服务器或平台。本项目不是这个；这里的 runtime 是 Codex 本身。

## Use Cases

Agent Team 最适合个人开发和小团队软件项目：你希望 AI 帮你开发，但不想失去顺序、review 质量和 release 安全。

适合的场景：

- 个人项目和小团队仓库
- MVP、原型、小型 SaaS 产品和 Web App
- 给现有项目加功能
- bugfix 和小范围重构
- 前端 UI/UX 改进
- API、auth、权限和隐私相关改动
- 发布前的 QA、Code Review 和 Security Review

最合适的任务大小，是大约 30 分钟到 2 天内可以完成的 feature、fix 或 refactor。更大的工作应该由 Lead 拆成更小的 gated tasks。

## Workflow

普通软件任务的默认流程是：

```text
Lead
 -> Product Designer
 -> Project Explorer
 -> Architect
 -> DevOps Engineer, early read-only release feasibility
 -> User Approval
 -> Frontend Engineer / Backend Engineer / DevOps Engineer
 -> Implementation Freeze
 -> QA Engineer / Code Reviewer / Security Reviewer
 -> Lead Final
```

重要规则：

- Implementation 在你明确批准之前会被阻塞。
- Fast path 可以缩短规划，但不能跳过 user approval。
- QA、Code Review 和 Security 可以在 Implementation Freeze 后并行运行。
- Release status 必须等待所有 required review artifacts。
- Security Reviewer 是 hard release gate，并且可以 veto release。

## Roles

- `Product Designer`：定义 user goal、scope、acceptance criteria、UX flow 和 UI states。
- `Project Explorer`：读取仓库，报告 facts、existing patterns、commands、dependencies 和 constraints。
- `Architect`：创建 technical plan、contracts、tradeoffs、test strategy、rollback plan，并在需要时验证 current external facts。
- `Frontend Engineer`：实现批准范围内的 UI、client behavior、styling 和 frontend tests。
- `Backend Engineer`：实现批准范围内的 APIs、server logic、data handling、permissions 和 backend tests。
- `DevOps Engineer`：早期检查 release/runtime feasibility；只有批准后才实现 CI/CD、deployment、env 和 runtime changes。
- `QA Engineer`：基于 evidence 验证 behavior 和 UX。QA 是 read-only，会向 Lead 请求缺失的 test/build/browser evidence。
- `Code Reviewer`：独立审查 correctness、maintainability、regressions 和 missing tests。
- `Security Reviewer`：独立审查 auth、permissions、data、secrets、dependencies、deployment risk 和 release security。

## Subagent Mode

Agent Team 使用一种执行模型：Subagent Mode。

```text
Lead dispatches each role with a bounded Role Packet.
Lead validates the Role Packet before dispatch.
The role runs as a specialist subagent.
Each role returns an artifact.
Lead validates the artifact before a gate can pass.
Gates and loops consume artifacts, not private reasoning.
```

Codex subagent 配置位于：

```text
.codex/config.toml
.codex/agents/*.toml
.codex/agents/README.md
```

Lead 不是 subagent。Lead 是 parent orchestrator。

如果 Codex subagent threads 不可用，Lead 必须报告 `SUBAGENT_UNAVAILABLE`，并询问是启用 subagents、对本次任务接受 reduced independence，还是取消。Reduced-independence execution 是明确 waiver，不是正常 Subagent Mode。

## Quick Start

把这些文件复制到任何你希望 Codex 遵循该 workflow 的项目根目录：

```text
AGENTS.md
agent-team-protocol/
.codex/
```

然后从该项目根目录开始一个新的 Codex session，并正常对话：

```text
Help me add login.
Review whether this branch is ready to release.
Refactor the dashboard export flow.
```

Codex 应该把你当作在和 `Lead` 对话，然后通过 protocol 路由工作。

复制/安装说明见 [INSTALL.md](INSTALL.md)。

## Quality Rules

这个协议会在错误代价较高的地方增加检查，但不会把每个小任务都变成重流程。

- Lead 为每个 role 打包 allowed inputs、forbidden inputs、evidence included、evidence missing 和 expected output。
- Role Packet 和返回的 artifact 必须通过 validity checks，Lead 才能 dispatch 或把 gate 判为 passed。
- Context budget 减少重复上下文，但不减少 gates、reviewer independence、approval 或 release checks。
- Architect 区分 repository facts、verified external facts、assumptions 和 recommendations。
- Engineers 以小的 implementation slices 工作，并记录 checks 和 rollback notes。
- QA 验证 evidence，而不是相信 implementation claims。
- Code Reviewer 对最有风险的 correctness claim 做简短 doubt check。
- Security Reviewer 把重要区域标记为 `Reviewed`、`Not relevant` 或 `Not checked`。
- 同类 blocking failure 连续 2 次未解决后停止 loop。

## Not A Runtime

Agent Team 是给 Codex 使用的 Markdown 规则系统，不是自定义 agent runtime。

它不包含：

- server
- database
- queue
- vector memory
- background workers
- Codex 之外的自动 multi-agent orchestration

真正的 runtime 系统适合产品化 agent platform、CI bot、长期后台任务、持久 memory、logs、evals 或自动任务执行。这个项目刻意更轻：把 `AGENTS.md`、`agent-team-protocol/` 和 `.codex/` 复制到项目里，然后正常和 Codex 协作。

## File Map

```text
AGENTS.md
  Loader file。保持简短。它告诉 Codex 要读取哪些 protocol files。

agent-team-protocol/
  gates、artifacts、approval、loops、boundaries 和 release policy 的主要规则。

agent-team-protocol/roles/
  每个 role 一个文件。把 role-specific rules 放在这里，而不是放进 AGENTS.md。

agent-team-protocol/examples/
  在真实仓库中测试该 protocol 的 trial workflows。

.agent-team/runs/
  可选的临时 role packets、artifacts、logs 和 patches。默认被 git 忽略，除非用户明确要求保留。

.codex/
  Codex custom agent definitions。包含 `.codex/agents/README.md`，用于把 agent names 对应到 role specs。

INSTALL.md
  复制/安装到另一个仓库的说明。
```

## Ask-User Policy

Codex 可以分析、检查并提出计划，但 implementation 需要用户明确批准。

协议会在以下情况继续前询问用户：

- scope 或 acceptance criteria 不清楚
- 需要 product tradeoff
- 引入新的 dependency、paid service、external account 或 vendor lock-in
- database schema、migrations、auth、permissions 或 privacy behavior 发生变化
- deployment、CI/CD、environment variables、secrets、cron、workers 或 cloud resources 发生变化
- 删除功能、重写模块，或需要 broad refactor
- reviewers 对 release readiness 有分歧
- Security 返回 `RELEASE_OK_WITH_RISK_ACCEPTANCE` 或 `RELEASE_BLOCKED`
- 任务明显超出原始 scope 或 budget

如果触发以上任一条件，就不允许 fast path。

## Loop Rules

工作失败后，workflow 不会每次都从零开始。Lead 会把工作送回最近的 responsible gate。

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

Same blocking failure repeated 2 consecutive times
 -> stop
 -> Lead reports what failed, what was tried, remaining options, and recommended next step
 -> third attempt requires explicit user approval
```

## Maintainer Notes

维护这个仓库时，请保持这些区别清楚：

- `AGENTS.md` 是 Codex 首先看到的小型 loader。
- `agent-team-protocol/` 保存稳定的 protocol rules。
- `.codex/` 保存 Subagent Mode 使用的 Codex custom agent definitions。
- `INSTALL.md` 说明如何把协议复制到另一个仓库。

不要把完整 protocol 复制进 `AGENTS.md`；重复会导致后续维护 drift。

## Design References

本协议受到 OpenAI、Anthropic 和 agent-skills 公开 agent architecture / engineering workflow patterns 的影响：

- [OpenAI Codex Subagents Setup](https://developers.openai.com/codex/subagents)
- [OpenAI Codex Subagents](https://developers.openai.com/codex/concepts/subagents)
- [OpenAI Agents SDK Handoffs](https://openai.github.io/openai-agents-python/handoffs/)
- [Anthropic Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)
- [agent-skills: Orchestration Patterns](https://github.com/addyosmani/agent-skills/blob/main/references/orchestration-patterns.md)

本协议把这些思想组合成一个可移植到任何仓库的 Markdown system：sub-agent style role isolation、artifact handoff、explicit gates、user approval、review independence、security veto 和 recovery loops。
