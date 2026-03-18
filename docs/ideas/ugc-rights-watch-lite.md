# 🛡️ UGC Rights Watch Lite

## 概要
UGC広告で使うBGM/素材の利用期限や利用範囲を追跡し、**「消すべき投稿」「再許諾すべき素材」**を通知する軽量リスク管理SaaS。

## 海外事例分析
- **Epidemic Sound / Artlist**: 商用ライセンス利用が拡大。
- 海外D2Cでは著作権クレーム対策ツール需要が増加。
- 日本では小規模運用で台帳管理がExcel止まり。

## ターゲット
- UGC運用代行
- 小規模EC
- 1人マーケ担当

## 料金
- Lite: $6/月（100素材）
- Team: $14/月（500素材）

## ユーザーフロー
1. 素材ライセンス情報を登録
2. 投稿URLと紐づけ
3. 期限30/7/1日前に通知
4. 期限切れ投稿一覧を出力

## デザインコンセプト
「信号機UI」。緑/黄/赤で期限ステータスを一目表示。

## アーキテクチャ
- Next.js管理画面
- Supabase
- Cron + Discord/メール通知
- S3に証跡添付

## DB設計
- users(id, plan)
- assets(id, user_id, name, license_type, expires_at)
- posts(id, user_id, platform, post_url, published_at)
- mappings(id, asset_id, post_id)
- alerts(id, user_id, asset_id, alert_level, sent_at)

## コスト見積もり（月）
- Supabase: $0〜$25
- Cron/通知: $1
- Hosting: $0
- 合計: 約$1〜$6

## MVPスコープ
- 素材台帳
- 投稿紐付け
- 期限通知
- CSVエクスポート

## マーケ計画
- UGC運用代行会社に直接DM
- 「ライセンス事故の事例」コンテンツで流入

## 技術スタック
Next.js, Supabase, AWS S3, Resend/Discord Webhook

## リスク
- 公式API制限 → 手動登録前提で開始
- 法務責任誤解 → 免責表記を明確化

## 競合分析
- 大手DAMツール: 高価格
- 国内: 汎用台帳のみ
- 差別化: **UGC投稿紐付け特化の低価格**

## $20達成シナリオ
- Lite $6 × 4社 = $24

## ユニットエコノミクス
- ARPU: $6
- 変動費/顧客: $0.2
- 粗利: $5.8 (96.6%)
