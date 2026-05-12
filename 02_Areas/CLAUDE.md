# 02_Areas — 継続的な責任領域

「終わりがない、維持し続ける情報」を置きます。プロジェクト横断で参照されます。

## サブ領域

| ディレクトリ | 内容 |
| --- | --- |
| `companies/<会社略称>/` | 顧客企業ごとの基本情報・組織図・過去取引・関係性 |
| `contacts/` | 人物カード（1人1ファイル）。`<会社略称>_<氏名>.md` |
| `sales-pipeline/` | 全案件の俯瞰（ダッシュボード的Markdown） |
| `our-company/` | 自社情報。プロダクト・価格・差別化・営業の前提知識 |

## 会社情報のフロントマター

```yaml
---
id: company-<会社略称>
type: company
legal_name: <正式社名>
industry: <業種>
size: <従業員規模>
website: <URL>
contacts: [<contact-id>, ...]
projects: [<project-id>, ...]   # 過去含む全案件
status: prospect | customer | churned
---
```

## 人物カードのフロントマター

```yaml
---
id: contact-<会社略称>-<氏名ローマ字>
type: contact
name: <氏名>
company: <会社ID>
title: <役職>
role: decision_maker | champion | user | gatekeeper
email: <email>
notes_summary: <一行サマリ>
---
```

## 運用ルール

- 同じ会社の情報を複数箇所に書かない（`companies/<会社>/` を単一の真実とする）
- 個別商談の話は **`01_Projects/` 側に書く**。Areasには「人」「会社」「自社」の **継続情報** のみ
