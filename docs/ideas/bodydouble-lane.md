# 🧍 BodyDouble Lane

## 概要
BodyDouble Laneは、パーソナル生産性（非同期ボディダブル）を狙う月$20到達向けマイクロプロダクト。**AIエージェント実装前提**で2週間MVPを組み、少人数課金で黒字化する。

## 海外事例分析
- 参照: Focusmate / Caveday / Study-with-me文化
- 海外での勝ち筋: 「軽い入力→すぐ共有できる出力」「テンプレ×コミュニティ拡散」「モバイル前提UI」
- 日本ローカライズ余地: 日本語フォント/敬語トーン/LINE・X導線の最適化

## ターゲット
- 主要: 在宅ワーカー、ADHD傾向の個人、受験生
- 課題: 既存ツールは重い・高い・英語前提で、日次運用に乗らない

## 料金
- Free / Plus $4
- 目標ARPU: $4〜$8

## ユーザーフロー
1. 30秒オンボード（用途選択）
2. 入力（URL/テキスト/画像/招待など）
3. AI処理（10〜60秒）
4. シェア・保存・再利用
5. 週次レポートで継続導線

## デザインコンセプト
- 「1画面1意思決定」
- 彩度高めアクセント + 大きめタイポ + スマホ先行
- 作成物を“見せたくなる”カードUI

## アーキテクチャ
- Front: Next.js (App Router) + Tailwind
- API: Next.js Route Handler / Cloudflare Workers（低コスト）
- DB: Supabase Postgres
- Queue: Upstash Redis / QStash
- Auth: Clerk or Supabase Auth
- AI: OpenAI gpt-4o-mini + 必要時画像/音声API

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, type, input_json, output_json, status)
- usage_events(id, user_id, action, tokens_in, tokens_out, cost_usd, created_at)
- subscriptions(id, user_id, stripe_customer_id, plan, status)
- shares(id, project_id, channel, short_url, clicks)

## コスト見積もり（月次）
- Hosting: $0〜$5（Vercel/CF無料枠優先）
- DB/Auth: $0（Supabase無料枠）
- AI: $2〜$12（無料枠＋miniモデル）
- その他: $0〜$3
- 合計: **$2〜$15**

## MVPスコープ
- Must: コア生成/保存/再編集/課金
- Should: テンプレ3種、1クリック共有
- Won't: 高度分析、チーム権限、ネイティブアプリ

## マーケ計画
- Day1: Xで「作例10本」投稿
- Week1: Product Hunt / Reddit / Discordコミュニティにデモ投下
- Week2: 事例note公開（数字付き）
- 紹介導線: 友達招待で1週間Pro

## 技術スタック
- Next.js / TypeScript / Supabase / Stripe / Upstash / OpenAI

## リスク
- 主リスク: 継続率の確保
- 対策: テンプレ固定、成功体験の短サイクル化、AI失敗時リトライと手動編集導線

## 競合分析
- 競合は高機能・高価格帯（$15〜$49）が中心
- 本案は「日本語最適化×軽量導線×低単価」で差別化

## $20達成シナリオ
- 例1: $5プラン × 4ユーザー = $20
- 例2: $7プラン × 3ユーザー = $21
- 目標到達期間: 1〜2ヶ月

## ユニットエコノミクス
- 想定ARPU: $6
- 変動費/ユーザー: $0.4〜$1.2
- 粗利率: 80〜93%
- 回収期間: 初月〜2ヶ月
