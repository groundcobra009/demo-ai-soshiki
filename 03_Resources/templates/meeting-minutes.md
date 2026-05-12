---
id: TEMPLATE-meeting-minutes
type: template
applies_to: meeting
---

# 議事録テンプレート（コピー用）

> このファイル自体は記入しない。コピーして `01_Projects/<project>/minutes/` に保存し、
> `id:` を新規採番、`TEMPLATE-` プレフィックスを削除すること。

```yaml
---
# meeting-YYYY-MM-DD-<会社略称>-<回次連番2桁>
id: meeting-YYYY-MM-DD-COMPANY-NN
type: meeting
date: YYYY-MM-DD
project: project-COMPANY-XXX
company: company-COMPANY
attendees:
  ours: []
  theirs: []
duration_min: 0
medium: zoom              # zoom | onsite | teams | phone
depends_on: []            # 前回議事録があれば
relates_to: []            # 関連retroなど
next_actions: []          # 後続スキルで埋まる
status: draft             # draft | review | final
tags: []
---
```

# <議題タイトル>

## 1. サマリ（3行以内）

## 2. 顧客課題（聞き取れた事実）

## 3. 顧客の現状（As-Is）

## 4. 提案・議論内容

## 5. 顧客の反応（一次情報のみ）
- 賛同点:
- 懸念点:

## 6. 決定事項

## 7. 次回アクション
| # | 内容 | 担当 | 期日 |
| - | ---- | ---- | ---- |

## 8. オープン論点（未決）

## 9. 引用（重要発言の原文）
