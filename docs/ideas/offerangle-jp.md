# 📦 OfferAngle JP

## 概要
OfferAngle JPは、マイクロSaaS（海外LP/広告から日本向け訴求角度を抽出）に特化した月$20到達を狙うマイクロプロダクト。

## 海外事例分析
- 参照潮流: Foreplay / AdSpy / SwipeWell の広告スワイプ文化
- 日本でのギャップ: 英語圏前提・UIが重い・共有導線が弱い
- 勝ち筋: 日本語UI + 3分以内で価値体験 + 共有導線最適化

## ターゲット
小規模D2C、広告運用者、LP制作者

## 料金
- プラン: Starter $9/月（50解析）
- 無料→有料転換導線: 初回の成功体験後に使用回数上限を提示

## ユーザーフロー
1. 海外LP URLを入力
2. 見出し/CTA/オファー構造を抽出
3. 日本向けの訴求角度3案を生成
4. Notion/CSVへエクスポート

## デザインコンセプト
分析ダッシュボードをカード化、比較が一目で見える。

## アーキテクチャ
Playwright scraper + HTML parser + LLM summarizer + export worker

## DB設計
- 主要テーブル: users, crawls, angle_cards, exports, subscriptions
- 監査/運用: created_at, updated_at, soft_deleteを全テーブルに付与

## コスト見積もり（月次）
- インフラ: $0〜$8（無料枠中心、超過時のみ従量）
- AIコスト: 約$0.18/ユーザー/月（抽出+翻案）
- その他: Stripe手数料のみ

## MVPスコープ
URL解析、角度3案、履歴保存、CSV出力、Stripe

## マーケ計画
Xで「今日の海外LP分解」無料投稿→導線

## 技術スタック
- Frontend: Next.js / React
- Backend: Serverless Functions
- DB: Supabase or DynamoDB
- Billing: Stripe
- Analytics: PostHog

## リスク
スクレイピング制限→手動貼り付けfallback。誤訳→根拠文を併記。

## 競合分析
手動調査より10倍速、一般翻訳よりマーケ文脈が強い。

## $20達成シナリオ
Starter 3人($27)で達成

## ユニットエコノミクス
ARPU $9 / 変動費 $0.7 / 粗利92%
