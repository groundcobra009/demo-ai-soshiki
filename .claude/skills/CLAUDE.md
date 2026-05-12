# .claude/skills/ — AI組織のスキル定義

Claude Code が自動認識するスキル群。`/<skill-name>` で起動できる。

## 命名・配置

- 各スキル1フォルダ: `.claude/skills/<kebab-case-name>/SKILL.md`
- フォルダ名 = スラッシュコマンド名（例: `transcript-to-minutes/` → `/transcript-to-minutes`）

## SKILL.md のフロントマター（公式仕様）

```yaml
---
name: <skill-name>                 # 省略可（ディレクトリ名がデフォルト）
description: <発動条件を書く>      # 推奨。Claudeはこれを見て自動発動を判断する
allowed-tools: Read Write Edit ... # スペース区切り or YAMLリスト
disable-model-invocation: false    # trueにすると手動 /name のみ
user-invocable: true               # falseで / メニューから隠す
context: fork                      # サブエージェントで実行する場合
agent: <subagent-name>             # context:fork時に指定
---
```

公式の必須キーは無く、`description` のみ強く推奨。

## 現在のスキル

| スキル | 目的 | 連携サブエージェント |
| --- | --- | --- |
| `transcript-to-minutes/` | 商談トランスクリプト → 構造化議事録 | `minutes-reviewer`（レビュー）/ `lost-deal-searcher`（類似事例） |

## 今後追加予定

- `next-action-router` — 議事録から次アクションを派生
- `proposal-writer` — 議事録 + 自社情報 → 提案書ドラフト
- `slide-writer` — 提案書 → スライド原稿
- `pipeline-summarizer` — 全案件の俯瞰を更新

## スキルとサブエージェントの使い分け

- **スキル** = 再利用可能な手順書。`/name` で呼ぶ。会話のメインコンテキストで実行
- **サブエージェント** = 独立コンテキストで動く専門家。`.claude/agents/<name>.md`。スキルから `context: fork` で呼べる

「ワークフロー」は独立概念ではなく、スキル本文の中に手順として書くか、複数スキルを組み合わせて表現する。
