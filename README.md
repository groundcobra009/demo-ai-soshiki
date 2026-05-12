# demo-ai-soshiki

ローカルで動く「AI営業組織」のナレッジベース構成デモ。
商談トランスクリプト → 議事録 → 依存関係 → 次商談・提案書・失注からの学び、
までを **1つのフォルダ構造** + **Claude Code の skills/agents** で回すことを目的とする。

> Anthropic 公式仕様（[`.claude/` ディレクトリ](https://code.claude.com/docs/en/claude-directory) / [Skills](https://code.claude.com/docs/en/skills) / [Subagents](https://code.claude.com/docs/en/sub-agents)）に準拠したリファレンス実装。

---

## このリポジトリで何が見られるか

- **PARAメソッド** に従ったナレッジ層の組み方（営業情報を Project / Area / Resource / Archive で整理する例）
- **Claude Code の `.claude/`** に置く skills と subagents の最小例
- **YAMLフロントマターによる依存関係表現**（`depends_on` / `relates_to` / `next_actions`）で、議事録 → 次会議・提案・失注retro を辿れる設計
- **ダミーの商談1件 + 失注事例1件** によるエンドツーエンドの動作イメージ

---

## 2層構造

このリポジトリは「ナレッジ層（PARA）」と「実行設定層（`.claude/`）」を明確に分けている。

### A. ナレッジ層（PARA） — 営業情報の保管

| ディレクトリ | 役割 |
| --- | --- |
| `01_Projects/` | 進行中の商談（締切・ゴールあり） |
| `02_Areas/` | 継続情報（会社・人物・自社情報・パイプライン全体） |
| `03_Resources/` | テンプレート・プレイブック・自社プロダクト情報 |
| `04_Archive/` | 完了案件（受注／失注、retro付き） |
| `inbox/` | 未整理のトランスクリプトを一時投入する場所 |

### B. 実行設定層（`.claude/`） — Claude Code が自動認識

| パス | 役割 |
| --- | --- |
| `.claude/skills/<name>/SKILL.md` | スキル（再利用可能な手順書）。`/<name>` で呼べる |
| `.claude/agents/<name>.md` | サブエージェント（独立コンテキストの専門家） |

「ワークフロー」は公式に独立概念は無く、**skill 本文の手順 + subagent への委譲** で表現する。

---

## ディレクトリ全景

```
demo-ai-soshiki/
├── .claude/
│   ├── agents/
│   │   ├── company-profiler.md       # 顧客企業プロファイルの整理・更新
│   │   ├── lost-deal-searcher.md     # 失注事例を横断検索し学びを抽出
│   │   └── minutes-reviewer.md       # 議事録ドラフトの品質レビュー
│   └── skills/
│       └── transcript-to-minutes/SKILL.md   # トランスクリプト → 議事録
├── CLAUDE.md                         # ナレッジベース全体の運用方針
├── 01_Projects/
│   └── 2026-Q2_テックフロンティア_SaaS導入/   ← ダミー進行中案件
│       ├── README.md
│       ├── lessons.md                # 失注事例からの学び転記
│       ├── transcripts/
│       │   └── 2026-04-15_01_初回ヒアリング.txt
│       └── minutes/
│           └── 2026-04-15_01_初回ヒアリング.md
├── 02_Areas/
│   ├── companies/テックフロンティア/
│   ├── contacts/{佐藤健一,山田美咲}.md
│   ├── our-company/                  # 自社情報
│   └── sales-pipeline/               # 案件俯瞰
├── 03_Resources/
│   └── templates/meeting-minutes.md
├── 04_Archive/
│   └── lost-deals/2026-Q1_Acme_AI導入/retro.md   ← ダミー失注事例
├── inbox/
└── chat-history.md                   # このリポジトリができるまでの会話履歴
```

---

## 含まれるスキルとサブエージェント

### Skill

| Skill | 起動 | 概要 |
| --- | --- | --- |
| [`transcript-to-minutes`](.claude/skills/transcript-to-minutes/SKILL.md) | `/transcript-to-minutes` | 商談トランスクリプトから構造化議事録を生成し `01_Projects/<案件>/minutes/` に保存 |

### Subagents

| Subagent | 委譲のタイミング |
| --- | --- |
| [`minutes-reviewer`](.claude/agents/minutes-reviewer.md) | 議事録ドラフトの品質チェック（事実と推測の分離、フロントマター完全性、次回アクションの具体性） |
| [`lost-deal-searcher`](.claude/agents/lost-deal-searcher.md) | 新規案件着手時に `04_Archive/lost-deals/` を横断し、類似失注事例からルールを抽出 |
| [`company-profiler`](.claude/agents/company-profiler.md) | 顧客企業のプロファイル整理・更新（Web情報も含む） |

### 連携イメージ

```
ユーザー: トランスクリプト投入
   ↓
/transcript-to-minutes  (.claude/skills/)
   ├─ minutes-reviewer     (.claude/agents/) で品質チェック
   ├─ lost-deal-searcher   (.claude/agents/) で類似事例検索 → relates_to に追記
   └─ 出力 → 01_Projects/<案件>/minutes/
```

---

## 依存関係の表現

全Markdownのフロントマターに以下のキーで関係性を持たせる:

```yaml
---
id: meeting-2026-04-15-techfrontier-01
type: meeting                       # meeting | proposal | slide | company | contact | retro
date: 2026-04-15
project: project-techfrontier-saas-2026q2
company: company-techfrontier
depends_on: []                      # 上流（前提）
relates_to: [retro-acmecorp-2026q1] # 横の関連
next_actions: []                    # 下流（派生TODO・次会議）
status: draft | review | final
---
```

本文中では `[[id]]` 形式で他文書にリンクする。

---

## 試してみる

```bash
git clone https://github.com/groundcobra009/demo-ai-soshiki.git
cd demo-ai-soshiki
claude   # Claude Code で開く
```

`.claude/skills/` と `.claude/agents/` が自動認識されるので、以下のように使える:

- 「初回ヒアリングの議事録を見せて」→ ダミー議事録を読みに行く
- `/transcript-to-minutes` → スキルが起動
- 「テックフロンティア案件に類似する失注事例ある？」→ `lost-deal-searcher` が委譲される

---

## ダミーデータについて

含まれる会社名・人物名・案件名は **すべて架空のダミー** である:

- 株式会社テックフロンティア（架空、進行中案件のダミー）
- Acme Corp（架空、失注事例のダミー）
- 佐藤 健一 / 山田 美咲（架空のキーパーソン）

実在の企業・個人とは無関係。

---

## ロードマップ

- `next-action-router` — 議事録から次アクション（次会議・提案書・社内タスク）を派生
- `proposal-writer` — 議事録 + 自社情報 → 提案書ドラフト
- `slide-writer` — 提案書 → スライド原稿
- `pipeline-summarizer` — 全案件の俯瞰を更新

---

## 参考

- [Anthropic — Skills](https://code.claude.com/docs/en/skills)
- [Anthropic — Subagents](https://code.claude.com/docs/en/sub-agents)
- [Anthropic — .claude directory](https://code.claude.com/docs/en/claude-directory)
- PARA Method: Projects / Areas / Resources / Archive

---

## 立ち上げ経緯

このリポジトリの構築プロセス（初回PARA構築 → 公式仕様の検証 → `.claude/` 構造への移行）は [`chat-history.md`](./chat-history.md) に記録してある。
