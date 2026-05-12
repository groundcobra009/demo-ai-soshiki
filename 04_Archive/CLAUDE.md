# 04_Archive — 完了案件

クローズした案件をフォルダごと移動して保管します。**削除しない**。
新規案件着手時にAIが横断検索して学びを抽出する重要資産です。

## サブ領域

| ディレクトリ | 内容 |
| --- | --- |
| `won-deals/` | 受注した案件 |
| `lost-deals/` | 失注した案件 |

## クローズ時のチェックリスト

1. `01_Projects/<案件>/README.md` の `status` を `won` or `lost` に更新
2. 案件フォルダ直下に `retro.md` を作成（下記テンプレート）
3. フォルダごと `04_Archive/won-deals/` or `lost-deals/` に移動
4. `02_Areas/companies/<会社>/README.md` の `projects:` リストを更新

## retro.md のテンプレート

```yaml
---
id: retro-<案件略称>
type: retro
project: <project-id>
outcome: won | lost
amount_jpy: <最終金額>
closed_at: YYYY-MM-DD
key_factors: [<要因タグ>]      # 例: price, timing, competitor, fit
lessons_for: [<同業種タグ>]    # 他案件に流用すべきタグ
---
```

本文には:
- **何が起きたか**（事実、時系列）
- **なぜそうなったか**（仮説、複数可）
- **次回以降のルール**（具体的な打ち手 / 避けるべき言動）
