# 🛡️ ConsentLabel Lite

## 概要
UGCやインタビュー素材に「公開先・期限・再利用範囲」をラベル化し、投稿前に**利用可否を色で判定**する軽量保険型SaaS。

## 海外事例分析
- **Frame.io**: クリエイティブ承認フローの定着
- **DocuSign**: 電子同意の当たり前化
- **Airtable運用文化**: 権利管理を表計算で回している現場が多い

## ターゲット
- UGC運用する小規模EC
- SNS運用代行の個人/小チーム

## 料金
- Lite: $6/月（3プロジェクト）
- Team: $12/月（無制限）

## ユーザーフロー
1. 素材アップロード
2. 同意条件をフォーム入力
3. ラベル自動発行
4. 投稿URLを入れると公開可否チェック

## デザインコンセプト
「**Traffic Label**」: 緑/黄/赤の即時判定で迷わせない。

## アーキテクチャ
- Next.js
- Supabase
- OCR（Tesseract）
- ルールエンジン（Node）

## DB設計
- users(id, plan)
- assets(id, user_id, type, hash)
- consents(id, asset_id, channels, expires_at, scope)
- checks(id, asset_id, target_channel, result)

## コスト見積もり（月）
- Supabase: $0〜$5
- OCR/処理: $1
- 合計: **$6以下**

## MVPスコープ
- 同意ラベル登録
- 投稿前チェック
- 期限切れ通知

## マーケ計画
- UGC運用代行向けチェックリスト配布
- 「事故防止テンプレ」無料公開

## 技術スタック
Next.js, Supabase, Node rule engine, Tesseract

## リスク
- 法務解釈の誤認 → 法律助言でない旨を明記
- 入力漏れ → 必須項目バリデーション

## 競合分析
- DocuSign: 重く高い
- スプレッドシート運用: チェック漏れが起こる

## $20達成シナリオ
- Team 2社 = $24

## ユニットエコノミクス
- ARPU: $9
- 変動費: $0.5
- 粗利率: **94%**
