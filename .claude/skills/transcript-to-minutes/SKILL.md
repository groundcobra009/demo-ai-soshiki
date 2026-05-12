---
name: transcript-to-minutes
description: 商談トランスクリプト（録音書き起こし）から構造化された議事録Markdownを生成し、01_Projects/<案件>/minutes/ に保存する。トランスクリプトファイルが渡された時、または「議事録を作って」「minutes化して」と言われた時に発動。
allowed-tools: Read Write Edit Glob Grep Bash(ls *) Bash(date *)
---

# transcript-to-minutes

商談のトランスクリプト（書き起こし）から、構造化された議事録Markdownを生成する。

## 入力

- トランスクリプトファイル（`.txt` / `.md` / `.vtt` 等）
- 対象プロジェクトID（不明なら本文から推定 → ユーザーに確認）
- 既存の `02_Areas/companies/<会社>/README.md`（参加者・組織情報の参照用）

## 出力

`01_Projects/<プロジェクトID>/minutes/YYYY-MM-DD_<回次2桁>_<議題>.md`

### 議事録のフォーマット

```yaml
---
id: meeting-YYYY-MM-DD-<会社略称>-<回次2桁>
type: meeting
date: YYYY-MM-DD
project: <project-id>
company: <company-id>
attendees:
  ours: [<contact-id or 氏名>]
  theirs: [<contact-id>]
duration_min: <分>
medium: zoom | onsite | teams | phone
depends_on: [<前回議事録ID>]
relates_to: []
next_actions: []
status: draft
tags: [<業種>, <フェーズ>]
---
```

本文セクション（順序固定）:

1. サマリ（3行以内）
2. 顧客課題（聞き取れた事実）
3. 顧客の現状（As-Is）
4. 提案・議論内容
5. 顧客の反応（賛同点 / 懸念点）
6. 決定事項
7. 次回アクション（# / 内容 / 担当 / 期日 の表）
8. オープン論点（未決）
9. 引用（重要発言の原文）
10. 補足: 仮説（任意。事実と分離）

詳細な見本は `01_Projects/2026-Q2_テックフロンティア_SaaS導入/minutes/2026-04-15_01_初回ヒアリング.md` を参照。

## 手順

1. **入力確認**: トランスクリプトのパスと日付を確認。プロジェクト不明なら冒頭から会社名を抽出し、`02_Areas/companies/` で照合
2. **ID採番**: `meeting-YYYY-MM-DD-<会社略称>-<回次2桁>` を生成。Glob で `01_Projects/**/minutes/*.md` を検索して衝突確認
3. **構造化抽出**: 上記フォーマットの各セクションを埋める。**事実と解釈を分離**:
   - 「顧客課題」「顧客の反応」「決定事項」には発言そのもの・直接観測可能な事実のみ
   - 推測・仮説は本文に混ぜず、末尾の「補足: 仮説」セクションへ
4. **引用**: 重要発言は「9. 引用」に **原文ママ** で残す（要約しない）
5. **依存関係の更新**:
   - 同プロジェクトの直前議事録があれば `depends_on:` に追加
   - 関連する `04_Archive/lost-deals/**/retro.md` で類似タグがあれば `relates_to:` に追加
6. **保存**: `01_Projects/<project>/minutes/` に保存。プロジェクトREADMEの `next_actions:` も更新
7. **品質チェック**: 下記の品質基準で自己レビュー
8. **レビュー依頼（任意）**: `minutes-reviewer` サブエージェントを呼び、品質チェックを通す

## 品質基準（生成後のセルフチェック）

- [ ] フロントマターの必須キー（id / type / date / project / company / attendees / status）が全て埋まっている
- [ ] 「顧客課題」と「決定事項」が混ざっていない
- [ ] 「次回アクション」は **担当と期日** が明記されている
- [ ] 推測・解釈は本文に混ぜず、「補足: 仮説」セクションに分離
- [ ] 引用は原文ママ（要約していない）
- [ ] 既存IDと衝突していない
- [ ] プロジェクトREADMEの `next_actions:` と整合している

## 失敗例（避けるべき）

- ❌ 顧客が言っていない要件を「課題」に書く
- ❌ 「予算: おそらく500万」のような曖昧な推測を断定形で書く
- ❌ 次回アクションを「検討する」で終わらせる（誰が・いつ・何を、まで書く）
- ❌ プロジェクトREADMEを更新せず議事録だけ作って終わる

## 関連

- 出力後にレビューを通したい場合: `.claude/agents/minutes-reviewer.md`
- 類似失注事例を盛り込みたい場合: `.claude/agents/lost-deal-searcher.md`
