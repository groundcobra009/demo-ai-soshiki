# 01_Projects — 進行中の商談

ここには **能動的に進めている商談・案件** のみを置きます。
ゴール（受注 or 失注）が確定したら `04_Archive/` に移動してください。

## 各プロジェクトフォルダの構造

```
<プロジェクトID>/
├── README.md         # 案件サマリ（フロントマター必須）
├── transcripts/      # 録音書き起こし（生データ）
├── minutes/          # 議事録（transcriptsから生成）
├── deliverables/     # 提案書・スライド原稿・見積等
└── lessons.md        # 着手時にArchiveから転記した学び（任意）
```

## README.md のフロントマター

```yaml
---
id: project-<会社略称>-<案件略称>
type: project
company: <会社ID>            # 02_Areas/companies/ 配下
status: active | stalled | closing
stage: lead | qualification | proposal | negotiation | closing
owner: <自社担当>
started_at: YYYY-MM-DD
target_close: YYYY-MM-DD
amount_jpy: <想定金額>
depends_on: []
relates_to: []
next_actions: []             # 次の会議や提案書のID
---
```

## 運用ルール

- 1案件1フォルダ。複数案件を混ぜない
- 議事録は `minutes/` に集約。`transcripts/` は触らない（生データ保護）
- 失注/受注時は `04_Archive/lost-deals/` or `won-deals/` へ **フォルダごと移動** し、
  `retro.md` を追加してから完了とする
