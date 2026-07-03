# Agent Team

Codex を、任意のリポジトリ内で gate 付きの AI ソフトウェア開発チームのように動かすための、移植可能な Markdown agent team です。

## Developer Note

このプロジェクトには、意図的にカスタム runtime を含めていません。runtime は Codex そのものです。`AGENTS.md` が Codex に読むべき内容を伝え、`agent-team-protocol/` が永続的なチームルールを保持します。

このリポジトリを保守するときは、次の区別を明確にしてください：

- `AGENTS.md` は Codex が最初に読む小さな loader です。
- `agent-team-protocol/` は Codex が次に読む詳細な protocol です。
- `INSTALL.md` は、この protocol を別のリポジトリへコピーする方法を説明します。

Codex 風の使い方のために、このリポジトリには AI が読める loader ファイルである [AGENTS.md](AGENTS.md) が含まれています。詳細ルールは `agent-team-protocol/` にあります。`AGENTS.md` と `agent-team-protocol/` の両方を対象リポジトリの root にコピーすると、通常どおり Codex と会話しながら、同じ team roles、gates、artifact handoff、loop rules、release checks に従わせることができます。

コピー/インストール手順は [INSTALL.md](INSTALL.md) を参照してください。

## Use Cases

Agent Team は、AI の支援を受けながらも順序、review quality、release safety を失いたくない個人開発や小規模チームのソフトウェア開発に向いています。

適した用途：

- 個人プロジェクトと小規模チームのリポジトリ
- MVP、プロトタイプ、小規模 SaaS、Web App
- 既存プロジェクトへの機能追加
- bugfix と focused refactor
- フロントエンド UI/UX 改善
- API、auth、permission、privacy 関連の変更
- QA、Code Review、Security Review を含む release-readiness check

理想的なタスクサイズは、およそ 30 分から 2 日で完了できる feature、fix、refactor です。より大きな作業は、Lead が小さな gated tasks に分割します。

## Not A Runtime

Agent Team は Codex のための Markdown ルールシステムであり、カスタム agent runtime ではありません。

含まれていないもの：

- server
- database
- queue
- vector memory
- background workers
- Codex 外での automatic multi-agent orchestration

本物の runtime system は、productized agent platform、CI bot、long-running background jobs、persistent memory、logs、evals、自動 task execution が必要な場合に有用です。このプロジェクトは意図的に軽量です：`AGENTS.md` と `agent-team-protocol/` をプロジェクトにコピーし、通常どおり Codex と作業します。

この設計は、この会話で定義した roles と gates に基づいています：

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

重要な考え方は artifact handoff です。Agents は自由にチャットしません。各 gate が構造化された artifact を書き、次の gate は必要な artifact だけを読みます。

## Engineering Model

この protocol は、workflow gates と組み合わせた sub-agent orchestration model を使います。

実際には次のように動きます：

- ユーザーは Lead とだけ話します。
- Lead は orchestrator として動きます。
- 各 role は specialist sub-agent のように動きます。
- 各 specialist は、許可された入力だけを受け取ります。
- 各 specialist は、自由形式のグループチャットに参加するのではなく artifact を返します。
- Review roles は implementation reasoning を信じるのではなく、evidence を独立して判断します。

実際の multi-agent runtime が利用できる場合、各 role は独自の context window で実行されるべきです。

Codex conversation が 1 つしか使えない場合、この protocol は `agent-team-protocol/09-context-boundaries.md` によって sub-agent 分離をシミュレートします。Lead は各 role に allowed inputs をパッケージし、各 role は別 role の private reasoning、無関係な transcript history、未承認の scope changes などの forbidden inputs を無視しなければなりません。

このチームは Agent Team 風の協調も使いますが、制御された artifacts、gates、loops を通してのみ行います：

- artifacts は decisions と evidence を保存します
- gates は作業のスキップを防ぎます
- loops は失敗した作業を最も近い responsible role に戻します
- QA、Code Review、Security は独立した release checks のままです

結果として、これは自由チャット型の agent swarm ではありません。より近いのは次の形です：

```text
Lead orchestrator
 -> isolated specialist sub-agents
 -> artifact handoff
 -> gated review
 -> repair loops
 -> final release decision
```

## Quality Rules

この protocol は、追加の厳密さを軽量に保ちます。小さなタスクすべてに重い ceremony を要求するわけではありませんが、失敗コストが高い場所ではより強い checks を追加します：

- Lead は各 role に対して、allowed inputs、evidence included、evidence missing、expected output をパッケージします。
- Architect は repository facts、verified external facts、assumptions、recommendations を分けます。
- Architect は verified external facts について、source URL、date checked、confidence、supported decision を記録します。
- Engineers は小さな implementation slices で作業し、checks と rollback notes を残します。
- UI が関係し、tools が利用できる場合、QA は browser behavior、console errors、core interactions を確認します。
- Code Reviewer は最もリスクの高い correctness claim に対して短い doubt check を使います。
- Security Reviewer は重要領域を `Reviewed`、`Not relevant`、`Not checked` として記録します。
- must-ask 条件に該当する場合、fast path は禁止されます。
- 同じ blocking failure が 2 回連続で未解決の場合、loop を停止します。

これらのルールは、すべてのタスクを重いプロセスにせず、独立性と evidence quality を高めるためのものです。

## Quick Start

Codex にこの team protocol に従わせたいプロジェクトの root に、次をコピーします：

```text
AGENTS.md
agent-team-protocol/
```

その後、そのプロジェクト root から新しい Codex session を開始し、通常どおり話します：

```text
Help me add login.
Review whether this branch is ready to release.
Refactor the dashboard export flow.
```

Codex は、あなたが Lead に話しているものとして扱い、その後 protocol gates を通して作業をルーティングします。

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

Implementation は user approval までブロックされます。

Security Reviewer は hard release gate であり、release を veto できます。

Implementation 中、この protocol は分離された engineering handoffs を期待します：

```text
06a-frontend-implementation.md
06b-backend-implementation.md
06c-devops-implementation.md
06-implementation-summary.md
```

QA、Code Reviewer、Security Reviewer は、これらの artifacts を独立して読みます。

## File Map

```text
AGENTS.md
  Loader file。短く保ちます。Codex に読むべき protocol files を伝えます。

agent-team-protocol/
  gates、artifacts、approval、loops、release policy の主要ルール。

agent-team-protocol/roles/
  role ごとに 1 ファイル。role-specific rules は AGENTS.md ではなくここに追加します。

agent-team-protocol/examples/
  実際のリポジトリで protocol をテストするための trial workflows。

agent-team-protocol/artifacts/
  任意の一時 task artifacts。ユーザーが明示的に保存を求めない限り、git では無視されます。

INSTALL.md
  別リポジトリへのコピー/インストール手順。
```

## Ask-User Policy

protocol は、次の場合に続行前にユーザーへ確認します：

- Scope または acceptance criteria が不明確な場合。
- Product tradeoff が必要な場合。
- 新しい dependency、paid service、external account、vendor lock-in を導入する場合。
- Database schema、migrations、auth、permissions、privacy behavior が変わる場合。
- Deployment、CI/CD、environment variables、secrets、cron、workers、cloud resources が変わる場合。
- 機能削除、module rewrite、broad refactor が必要な場合。
- Reviewers が release readiness について不一致の場合。
- Security が `RELEASE_OK_WITH_RISK_ACCEPTANCE` または `RELEASE_BLOCKED` を返した場合。
- タスクが元の scope または budget を大きく超える場合。

これらの条件のいずれかに該当する場合、fast path は使えません。

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

同じ blocking failure が 2 回連続で未解決
 -> stop
 -> Lead が失敗内容、試したこと、残りの選択肢、推奨する次の一手を報告
 -> 3 回目の試行にはユーザーの明示的な承認が必要
```

## Design References

この protocol は、OpenAI、Anthropic、agent-skills が公開している agent architecture と engineering workflow patterns の影響を受けています：

- [OpenAI Codex Subagents](https://developers.openai.com/codex/concepts/subagents)
- [OpenAI Agents SDK Handoffs](https://openai.github.io/openai-agents-python/handoffs/)
- [Anthropic Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)
- [agent-skills: Orchestration Patterns](https://github.com/addyosmani/agent-skills/blob/main/references/orchestration-patterns.md)

この protocol は、これらの考え方を repo-portable な Markdown system として組み合わせています：sub-agent style role isolation、artifact handoff、explicit gates、user approval、review independence、security veto、recovery loops。
