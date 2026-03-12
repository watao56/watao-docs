# ☕ Handoff Cafe

## 概要
Handoff Cafeは、ソーシャル/コミュニティ（15分相互フィードバック交換）に特化した月$20到達を狙うマイクロプロダクト。

## 海外事例分析
- 参照潮流: Indie Hackersのroast文化 / Lunchclubのマッチング体験
- 日本でのギャップ: 英語圏前提・UIが重い・共有導線が弱い
- 勝ち筋: 日本語UI + 3分以内で価値体験 + 共有導線最適化

## ターゲット
個人開発者、デザイナー、副業クリエイター

## 料金
- プラン: 参加無料 / Host $6/月（週次ルーム3つ）
- 無料→有料転換導線: 初回の成功体験後に使用回数上限を提示

## ユーザーフロー
1. 今日の進捗を1枚投稿
2. AIが「見るべき観点」3つを自動付与
3. 他ユーザー2名に15分レビューを返す
4. レビュー交換が完了するとバッジ付与

## デザインコンセプト
カフェのメニュー風カードUI。レビューしやすい余白と短文導線。

## アーキテクチャ
Next.js + Supabase Auth/DB + Edge Functions + cron digest

## DB設計
- 主要テーブル: users, rooms, posts, reviews, match_logs, badges, plans
- 監査/運用: created_at, updated_at, soft_deleteを全テーブルに付与

## コスト見積もり（月次）
- インフラ: $0〜$8（無料枠中心、超過時のみ従量）
- AIコスト: 約$0.01/ユーザー/月（要約のみ）
- その他: Stripe手数料のみ

## MVPスコープ
投稿カード、2wayマッチ、レビューUI、週間ダイジェスト、課金

## マーケ計画
Discordコミュニティ提携、週1テーマ企画で定着

## 技術スタック
- Frontend: Next.js / React
- Backend: Serverless Functions
- DB: Supabase or DynamoDB
- Billing: Stripe
- Analytics: PostHog

## リスク
過疎化→週次テーマと自動再参加導線。荒れ対策→通報+自動隠し。

## 競合分析
Discordよりレビュー品質を担保、Slackより軽量。

## $20達成シナリオ
Hostプラン4人($24)で達成

## ユニットエコノミクス
ARPU $6 / 変動費 $0.12 / 粗利98%
