# チャットヒストリー

このリポジトリ（ローカルAI営業組織のナレッジベース）を立ち上げた一連のやり取りの記録。
日付: 2026-05-12 / 実行環境: Claude Code (Opus 4.7)

---

## 1. 初回リクエスト（ユーザー）

> 今からいわゆるローカルのAI組織みたいなのを作っていこうと思っています。まずは商談ごとの議事録を渡したら、トランスクリプトを渡したら議事録管理できて、それの依存関係を結んで次の商談につなげるとか、企画を作るとか、スライドを作るとか、他の商談、例えば失注した時の反省点に活かすとか、そういったものを何か管理していきたいと思ってます。フォルダの構成はディレクトリー構造、ドキュメントはパラメソッドでやって、プロジェクト、エリア、リソース、アーカイブというところでございますと。お客さんと商談中はプロジェクトに入っていくのかなと思ってます。こんな感じでディレクトリー構造を作って、さらにcloud.mdとかも作ってください。また会社情報とかどこに入れたらいいのか分かんないので、そこも箱として用意してください。まずはディレクトリー構造を作ってください。スキルは今後定義していきますが、まずはトランスクリプトから議事録を作るというのを作りたいと思ってます。最初ダミーデータで議事録とかそういったデータも作っていいかもしれませんね。

### 要件の解釈
- PARAメソッド（Projects / Areas / Resources / Archive）でディレクトリを構成
- 各層に `CLAUDE.md` を置いて運用ルールを記述
- 会社情報の置き場所を用意（自社／顧客）
- 「商談議事録 → 依存関係 → 次商談・企画・スライド・失注反省」のナレッジループを設計
- 最初のスキルは「トランスクリプト → 議事録」
- ダミーデータで動作イメージを示す

### 実装（Claudeの応答）

以下の構造を作成:

```
demo-ai-soshiki/
├── CLAUDE.md                              # 全体方針
├── 01_Projects/
│   ├── CLAUDE.md
│   └── 2026-Q2_テックフロンティア_SaaS導入/
│       ├── README.md
│       ├── lessons.md                     # Archiveから転記した学び
│       ├── transcripts/2026-04-15_01_初回ヒアリング.txt   ← ダミー
│       └── minutes/2026-04-15_01_初回ヒアリング.md        ← ダミー
├── 02_Areas/
│   ├── CLAUDE.md
│   ├── companies/テックフロンティア/README.md
│   ├── contacts/{佐藤健一,山田美咲}.md
│   ├── our-company/README.md              # 自社情報の箱
│   └── sales-pipeline/README.md           # 全案件俯瞰
├── 03_Resources/
│   ├── CLAUDE.md
│   └── templates/meeting-minutes.md
├── 04_Archive/
│   ├── CLAUDE.md
│   └── lost-deals/2026-Q1_Acme_AI導入/retro.md   ← ダミー失注
├── skills/                                # ← 後に .claude/skills/ に移動
│   ├── CLAUDE.md
│   └── transcript-to-minutes/SKILL.md
└── inbox/CLAUDE.md
```

設計のポイント:
- 依存関係は YAMLフロントマター（`depends_on` / `relates_to` / `next_actions`）+ 本文の `[[id]]` リンク
- 失注 → 新規案件への学習導線を `04_Archive/lost-deals/**/retro.md` → 新案件の `lessons.md` に明示
- 会社情報は `02_Areas/companies/<会社>/`、自社は `02_Areas/our-company/`、人物は `02_Areas/contacts/` に分離
- スキル形式: `skills/<名前>/SKILL.md` に「目的・入力・出力・手順・品質基準・失敗例」を記述

---

## 2. 検証依頼（ユーザー）

> `skills/` の階層をアンソロピックの公式をウェブサーチして、ここで正しいのか否か確認してください。なんとなく `.claude/` の下の階層にあって、サブエージェントとかワークフローとかっていうのを持ってた方がいいかなと思ってます。

### Web調査（Claude）

公式ドキュメント（docs.claude.com / code.claude.com）を確認した結果:

| 私の配置 | 公式の正解 |
| --- | --- |
| `skills/transcript-to-minutes/SKILL.md` | `.claude/skills/transcript-to-minutes/SKILL.md` |
| サブエージェントなし | `.claude/agents/<name>.md`（**単一の .md ファイル**、ディレクトリ形式ではない） |
| SKILL.md 独自フロントマター | 公式: `description` 推奨、`name` / `allowed-tools` / `disable-model-invocation` / `context: fork` / `agent: <name>` 等 |

公式ルール:
- **Skills** = `.claude/skills/<name>/SKILL.md`（ディレクトリ + 中に SKILL.md）
- **Subagents** = `.claude/agents/<name>.md`（単一ファイル）
- **「ワークフロー」は公式に独立概念なし**。skill 本文の手順 + subagent 委譲で表現
- スキルから `context: fork` + `agent: <subagent>` でサブエージェントを呼べる
- スコープ優先順位: Enterprise > Personal (`~/.claude/`) > Project (`.claude/`) > Plugin

参照ドキュメント:
- https://code.claude.com/docs/en/skills
- https://code.claude.com/docs/en/sub-agents
- https://code.claude.com/docs/en/claude-directory
- https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview

### 移行プランの承認（AskUserQuestion）

ユーザーは:
- 移行方針: **公式準拠に移行（推奨）**
- 最初に置くサブエージェント: **minutes-reviewer / lost-deal-searcher / company-profiler** を全て

### 実装（Claudeの応答）

1. `skills/` → `.claude/skills/` に移動
2. `SKILL.md` のフロントマターを公式仕様に書き換え（`description` 中心、`allowed-tools` 等を追加）
3. `.claude/agents/` を新設し、3つのサブエージェントを作成:
   - **minutes-reviewer** — 議事録の品質レビュー（事実と推測の分離、フロントマター完全性、次回アクションの具体性）
   - **lost-deal-searcher** — `04_Archive/lost-deals/` 横断検索で類似失注事例 → 学び抽出
   - **company-profiler** — 顧客企業のプロファイル整理・更新（Web情報も含む）
4. `.claude/skills/CLAUDE.md` と `.claude/agents/CLAUDE.md` を新設（公式仕様の説明付き）
5. ルート `CLAUDE.md` を更新し、「ナレッジ層（PARA）」と「実行設定層（`.claude/`）」の2層構造を明示

### 最終ディレクトリ構造

```
demo-ai-soshiki/
├── .claude/                               # 実行設定層（Claude Codeが自動認識）
│   ├── agents/
│   │   ├── CLAUDE.md
│   │   ├── company-profiler.md
│   │   ├── lost-deal-searcher.md
│   │   └── minutes-reviewer.md
│   ├── skills/
│   │   ├── CLAUDE.md
│   │   └── transcript-to-minutes/SKILL.md
│   └── settings.local.json
├── CLAUDE.md                              # ナレッジ層 全体方針
├── 01_Projects/                           # 進行中の商談
│   ├── CLAUDE.md
│   └── 2026-Q2_テックフロンティア_SaaS導入/
│       ├── README.md
│       ├── lessons.md
│       ├── transcripts/2026-04-15_01_初回ヒアリング.txt
│       └── minutes/2026-04-15_01_初回ヒアリング.md
├── 02_Areas/                              # 継続情報
│   ├── CLAUDE.md
│   ├── companies/テックフロンティア/README.md
│   ├── contacts/{佐藤健一,山田美咲}.md
│   ├── our-company/README.md
│   └── sales-pipeline/README.md
├── 03_Resources/                          # 参考資料
│   ├── CLAUDE.md
│   └── templates/meeting-minutes.md
├── 04_Archive/                            # 完了案件
│   ├── CLAUDE.md
│   └── lost-deals/2026-Q1_Acme_AI導入/retro.md
└── inbox/CLAUDE.md                        # 一時投入ボックス
```

---

## 3. 連携設計（参考）

```
ユーザー: トランスクリプト投入
   ↓
/transcript-to-minutes  (.claude/skills/)
   ├─ minutes-reviewer     (.claude/agents/) で品質チェック
   ├─ lost-deal-searcher   (.claude/agents/) で類似事例検索 → relates_to に追記
   └─ 出力 → 01_Projects/<案件>/minutes/
```

今後追加予定のスキル:
- `next-action-router` — 議事録から次アクションを派生
- `proposal-writer` — 議事録 + 自社情報 → 提案書ドラフト
- `slide-writer` — 提案書 → スライド原稿
- `pipeline-summarizer` — 全案件の俯瞰を更新

---

## 4. 共有

このリポジトリ立ち上げの記録 = 本ファイル `chat-history.md`。
共有相手にレビューしてもらう前提でパブリックリポジトリとして公開。
