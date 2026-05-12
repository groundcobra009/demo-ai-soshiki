# .claude/agents/ — サブエージェント定義

Claude Code が自動認識するサブエージェント。各 `.md` ファイル1つで1エージェント。

## 命名・配置

- `.claude/agents/<kebab-case-name>.md`（**単一の .md ファイル**。ディレクトリ形式ではない）
- ファイル名 = エージェント名

## フロントマター（公式仕様）

```yaml
---
name: <agent-name>
description: <委譲条件を書く。Claudeはこれを見て自動委譲を判断する>
tools: Read, Write, Edit, Glob, Grep   # カンマ区切り。省略すると全ツール許可
model: inherit | sonnet | opus | haiku # 省略可
---
```

本文（フロントマター以下）はそのサブエージェントの **システムプロンプト** になる。

## 現在のサブエージェント

| エージェント | 役割 |
| --- | --- |
| `minutes-reviewer.md` | 議事録ドラフトの品質レビュー |
| `lost-deal-searcher.md` | 失注事例を横断検索し新規案件に学びを提供 |
| `company-profiler.md` | 顧客企業のプロファイル整理・更新 |

## 呼び出し方

- Claudeが description に基づいて自動委譲
- スキルから明示的に呼ぶ: スキル側の frontmatter で `context: fork` + `agent: <name>`
- ユーザーが明示: 「<agent-name> を使って ...」と指示
