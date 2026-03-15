# 🛡️ DisputeSprint Lite

## 概要
Stripe/PayPalのチャージバック対応で必要な証跡（会話・納品・利用ログ）を**期限内に1ページで提出可能な形へ自動整理**する保険型マイクロSaaS。監視アラートではなく、対応作業の時短と勝率改善に寄せる。

## 海外事例分析
- Chargeflow / Midigator: 大型EC向け、価格と導入が重い
- Stripe標準UI: 管理は可能だが証跡整理が手作業
- 日本ギャップ: 個人SaaS/小規模EC向けの“軽量紛争対応”が不足

## ターゲット
- Stripe利用の個人SaaS開発者
- デジタル商材販売者
- 小規模EC運営

## 料金
- Free: 月1件まで
- Lite: $9/月（5件まで、テンプレ提出文）
- Pro: $19/月（30件、チーム共有）

## ユーザーフロー
1. 支払いIDを入力
2. 注文・利用履歴・会話ログを取り込み
3. AIが提出テンプレを生成
4. 不足証跡のチェックリスト表示
5. 提出用PDFをダウンロード

## デザインコンセプト
- **“Case Board”**
- 事件ボード風のカードUI
- 締切までの残時間を視覚化

## アーキテクチャ
- Next.js
- Ingest API (Webhook + CSV)
- LLM整形（mini）
- PDF生成（Playwright）
- DB: Postgres(Supabase)

## DB設計
- users(id, email, plan)
- cases(id, user_id, provider, payment_id, due_at, status)
- evidences(id, case_id, type, source, ref_url, meta_json)
- drafts(id, case_id, narrative_text, generated_at)
- exports(id, case_id, pdf_url, created_at)
- subscriptions(id, user_id, status)

## コスト見積もり（月）
- Hosting/Supabase: $0
- PDF生成: $1
- LLM整形: $2
- ストレージ: $1
- 合計: **約$4/月**

## MVPスコープ
- Stripe案件のみ
- 手動CSVインポート
- 提出文ドラフト + PDF出力
- 締切カウントダウン

## マーケ計画
- 「チャージバック提出1時間→10分」事例公開
- Stripe開発者コミュニティで配布
- 小規模EC向けテンプレ配布でリード獲得

## 技術スタック
Next.js / Supabase / Stripe API / OpenAI mini / Playwright / Resend

## リスク
- 法的助言との境界（リーガルディスクレーマ必須）
- 決済事業者API仕様変更

## 競合分析
- Chargeflow: 高機能・高価格 → **個人向け低価格**
- 手作業: 無料だが時間損失大 → **提出作業時間を短縮**

## $20達成シナリオ
- Lite 3人（$27 MRR）
- または Pro 2人（$38 MRR）

## ユニットエコノミクス
- ARPU: $11.0
- 変動費/人: $0.6
- 粗利/人: $10.4
- 粗利率: **94.5%**
