# Agent Team

Agent Team は、任意のリポジトリ内で Codex を規律あるソフトウェア開発チームのように動かすための、移植可能な Markdown protocol です。

あなたは通常どおり Codex と会話します。違いは、Codex がチーム workflow に従うことです：まず計画し、実装前に確認し、承認された scope 内で変更し、QA、Code Review、Security Review を通ってから release を推奨します。

## What This Is

このリポジトリは、Codex に再利用可能なチーム運用マニュアルを与えます。

平たく言うと：

1. あなたは `Lead` とだけ話します。
2. `Lead` は specialist subagents に bounded task packets を作ります。
3. Specialist subagents は structured artifacts を返します。
4. QA、Code Review、Security が evidence を独立して確認します。
5. `Lead` が結果をまとめ、release を推奨できるかを伝えます。

これは自由チャット型の agent swarm ではありません。Roles は気軽に会話せず、artifacts で作業を handoff します。

```text
User
 -> Lead
 -> specialist subagents
 -> artifacts
 -> gates and recovery loops
 -> final release decision
```

## Key Terms

- `Lead`：あなたが直接話す唯一の role。Lead は作業を route し、approval を求め、artifacts を検証し、最終状態をまとめます。
- `Subagent`：独立した context で動く専門 role。例：Architect、QA Engineer、Security Reviewer。
- `Gate`：workflow の checkpoint。次へ進む前に artifact または明確な decision が必要です。
- `Artifact`：構造化された handoff 結果。decisions、evidence、missing evidence、blockers、next action を記録します。
- `Role Packet`：Lead が subagent に渡す bounded task packet。objective、allowed inputs、forbidden inputs、evidence、expected output を含みます。
- `Implementation Freeze`：実装後の freeze point。QA、Code Review、Security が同じ diff または implementation artifact を確認します。
- `Security Veto`：Security Reviewer は `RELEASE_BLOCKED` で release を止められます。Lead はそれを上書きできません。
- `Runtime`：agents を自動実行する本物の server/platform。このプロジェクトはそれではありません。ここでの runtime は Codex そのものです。

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

## Workflow

通常のソフトウェア作業では、default flow は次の通りです：

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

重要なルール：

- Implementation は、あなたが明示的に承認するまで block されます。
- Fast path は planning を短くできますが、user approval は skip できません。
- QA、Code Review、Security は Implementation Freeze 後に parallel 実行できます。
- Release status は、すべての required review artifacts を待つ必要があります。
- Security Reviewer は hard release gate であり、release を veto できます。

## Roles

- `Product Designer`：user goal、scope、acceptance criteria、UX flow、UI states を定義します。
- `Project Explorer`：リポジトリを読み、facts、existing patterns、commands、dependencies、constraints を報告します。
- `Architect`：technical plan、contracts、tradeoffs、test strategy、rollback plan を作成し、必要に応じて current external facts を検証します。
- `Frontend Engineer`：承認された UI、client behavior、styling、frontend tests を実装します。
- `Backend Engineer`：承認された APIs、server logic、data handling、permissions、backend tests を実装します。
- `DevOps Engineer`：早期に release/runtime feasibility を確認します。CI/CD、deployment、env、runtime changes の実装は approval 後のみです。
- `QA Engineer`：evidence に基づいて behavior と UX を検証します。QA は read-only で、missing test/build/browser evidence は Lead に依頼します。
- `Code Reviewer`：correctness、maintainability、regressions、missing tests を独立して review します。
- `Security Reviewer`：auth、permissions、data、secrets、dependencies、deployment risk、release security を独立して review します。

## Subagent Mode

Agent Team は 1 つの実行モデルを使います：Subagent Mode。

```text
Lead dispatches each role with a bounded Role Packet.
Lead validates the Role Packet before dispatch.
The role runs as a specialist subagent.
Each role returns an artifact.
Lead validates the artifact before a gate can pass.
Gates and loops consume artifacts, not private reasoning.
```

Codex subagent configuration は次にあります：

```text
.codex/config.toml
.codex/agents/*.toml
.codex/agents/README.md
```

Lead は subagent ではありません。Lead は parent orchestrator です。

Codex subagent threads が利用できない場合、Lead は `SUBAGENT_UNAVAILABLE` を報告し、subagents を有効化するか、このタスクだけ reduced independence で続行するか、キャンセルするかを確認します。Reduced-independence execution は明示的な waiver であり、通常の Subagent Mode ではありません。

## Quick Start

Codex にこの workflow に従わせたいプロジェクトの root に、次をコピーします：

```text
AGENTS.md
agent-team-protocol/
.codex/
```

その後、そのプロジェクト root から新しい Codex session を開始し、通常どおり話します：

```text
Help me add login.
Review whether this branch is ready to release.
Refactor the dashboard export flow.
```

Codex は、あなたが `Lead` に話しているものとして扱い、その後 protocol を通して作業を route します。

コピー/インストール手順は [INSTALL.md](INSTALL.md) を参照してください。

## Quality Rules

この protocol は、失敗コストが高い場所に checks を追加しますが、小さなタスクすべてを重い ceremony にするわけではありません。

- Lead は各 role に対して、allowed inputs、forbidden inputs、evidence included、evidence missing、expected output をパッケージします。
- Role Packet と返却された artifact は validity checks を通過しない限り、Lead は dispatch したり gate を passed と扱ったりできません。
- Context budget は重複 context を減らしますが、gates、reviewer independence、approval、release checks は減らしません。
- Architect は repository facts、verified external facts、assumptions、recommendations を分けます。
- Engineers は小さな implementation slices で作業し、checks と rollback notes を残します。
- QA は implementation claims を信じるのではなく evidence を検証します。
- Code Reviewer は最もリスクの高い correctness claim に対して短い doubt check を使います。
- Security Reviewer は重要領域を `Reviewed`、`Not relevant`、`Not checked` として記録します。
- 同じ blocking failure が 2 回連続で未解決の場合、loop を停止します。

## Not A Runtime

Agent Team は Codex のための Markdown ルールシステムであり、カスタム agent runtime ではありません。

含まれていないもの：

- server
- database
- queue
- vector memory
- background workers
- Codex 外での automatic multi-agent orchestration

本物の runtime system は、productized agent platform、CI bot、long-running background jobs、persistent memory、logs、evals、自動 task execution が必要な場合に有用です。このプロジェクトは意図的に軽量です：`AGENTS.md`、`agent-team-protocol/`、`.codex/` をプロジェクトにコピーし、通常どおり Codex と作業します。

## File Map

```text
AGENTS.md
  Loader file。短く保ちます。Codex に読むべき protocol files を伝えます。

agent-team-protocol/
  gates、artifacts、approval、loops、boundaries、release policy の主要ルール。

agent-team-protocol/roles/
  role ごとに 1 ファイル。role-specific rules は AGENTS.md ではなくここに追加します。

agent-team-protocol/examples/
  実際のリポジトリで protocol をテストするための trial workflows。

.agent-team/runs/
  任意の一時 role packets、artifacts、logs、patches。ユーザーが明示的に保存を求めない限り、git では無視されます。

.codex/
  Codex custom agent definitions。agent names を role specs に対応させる `.codex/agents/README.md` を含みます。

INSTALL.md
  別リポジトリへのコピー/インストール手順。
```

## Ask-User Policy

Codex は analyze、inspect、plan proposal はできますが、implementation には明示的な user permission が必要です。

protocol は、次の場合に続行前にユーザーへ確認します：

- scope または acceptance criteria が不明確
- product tradeoff が必要
- 新しい dependency、paid service、external account、vendor lock-in を導入する
- database schema、migrations、auth、permissions、privacy behavior が変わる
- deployment、CI/CD、environment variables、secrets、cron、workers、cloud resources が変わる
- 機能削除、module rewrite、broad refactor が必要
- reviewers が release readiness について不一致
- Security が `RELEASE_OK_WITH_RISK_ACCEPTANCE` または `RELEASE_BLOCKED` を返す
- task が元の scope または budget を大きく超える

これらの条件のいずれかに該当する場合、fast path は使えません。

## Loop Rules

作業が失敗しても、workflow は毎回ゼロから再開しません。Lead は最も近い responsible gate に作業を戻します。

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

このリポジトリを保守するときは、次の区別を明確にしてください：

- `AGENTS.md` は Codex が最初に読む小さな loader です。
- `agent-team-protocol/` は永続的な protocol rules を保持します。
- `.codex/` は Subagent Mode 用の Codex custom agent definitions を保持します。
- `INSTALL.md` は、この protocol を別のリポジトリへコピーする方法を説明します。

完全な protocol を `AGENTS.md` に重複して書かないでください。重複は将来の drift を生みます。

## Design References

この protocol は、OpenAI、Anthropic、agent-skills が公開している agent architecture と engineering workflow patterns の影響を受けています：

- [OpenAI Codex Subagents Setup](https://developers.openai.com/codex/subagents)
- [OpenAI Codex Subagents](https://developers.openai.com/codex/concepts/subagents)
- [OpenAI Agents SDK Handoffs](https://openai.github.io/openai-agents-python/handoffs/)
- [Anthropic Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)
- [agent-skills: Orchestration Patterns](https://github.com/addyosmani/agent-skills/blob/main/references/orchestration-patterns.md)

この protocol は、これらの考え方を repo-portable な Markdown system として組み合わせています：sub-agent style role isolation、artifact handoff、explicit gates、user approval、review independence、security veto、recovery loops。
