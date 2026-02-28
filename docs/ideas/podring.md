# 👥 PodRing

## 概要
3〜5人の「実行ポッド」を作り、毎日25分の成果をスクショ1枚で報告するコミュニティSaaS。AIが進捗を要約して週報カードを自動生成。

## 海外事例分析
- **主要事例:** Focusmate / Caveday / Geneva
- **観察:** 海外では大規模SNS疲れから小規模クローズドコミュニティが伸長。
- **日本ローカライズ仮説:** 日本では匿名+小人数の心理安全性が重要。

## ターゲット
- 主: 副業ビルダー、受験生、資格学習者
- 副: ひとり起業家コミュニティ運営者
- JTBD: ひとりだと続かない作業を仕組みで継続したい

## 料金
- Free, Crew $5/月
- 目標は**月$20**なので、初期は3〜5人課金で到達可能な設計にする

## ユーザーフロー
1. 目的タグでポッド参加
2. 25分作業後に成果スクショ投稿
3. AIが当日サマリを生成
4. 週次で実行率ランキング表示

## デザインコンセプト
- キーワード: Ring, ritual, warm dark
- 共有したくなるUI: ポッドごとの「連続日数バッジ」表示
- MVP時点のUI要素: ポッド一覧、投稿タイムライン、週報カード

## アーキテクチャ
- Frontend: Next.js + Tailwind
- Backend: Next.js Route Handlers / Cloudflare Workers（軽量API）
- DB: Supabase Postgres
- Queue/Batch: GitHub Actions cron or EventBridge + Lambda（AWS利用時）
- Auth: Clerk or Supabase Auth
- Notification: Discord webhook / Email（Resend free）

## DB設計（MVP）
- `users(id, email, plan, created_at)`
- `projects(id, user_id, name, settings_json, created_at)`
- `events(id, project_id, type, payload_json, created_at)`
- `artifacts(id, project_id, url, meta_json, created_at)`
- `billing_events(id, user_id, amount_usd, source, created_at)`

## コスト見積もり（月次）
- Hosting: $0〜$5（Vercel/Cloudflare/AWS Free Tier）
- DB: $0（Supabase Free）
- AI: $1〜$2（テキスト要約のみ）
- 通知/メール: $0（無料枠）
- 合計: **$1.5〜$4**

## MVPスコープ（2週間）
- Must: ポッド作成/参加と日次投稿
- Must: 週報自動要約
- Should: 連続日数バッジ
- Won't: 複雑なチーム権限/多言語高度対応

## マーケ計画
- 初動: X/Discordで「作例」を毎日投稿
- ループ: ポッド週報をX共有→参加希望者が流入
- 獲得目標: 2週間で無料ユーザー30人、転換率10%で有料3人

## 技術スタック
- Next.js 15 / TypeScript / Tailwind / shadcn/ui
- Supabase Postgres / Prisma
- OpenAI gpt-4o-mini（要約・分類） or 画像系はReplicate/FLUX Schnell
- Stripe Payment Links

## リスク
- 初期コミュニティ過疎
- 荒らし投稿
- 対策: ユースケース限定 + 無料枠制御 + 監視ダッシュボード

## 競合分析
- 既存競合: Discord, Focusmate, Caveday
- 差別化: 「投稿+週報自動化」に絞り運営負荷を極小化

## $20達成シナリオ
- 価格: $5/月
- 必要有料ユーザー: 4人
- 目標到達時期: 1〜2ヶ月目

## ユニットエコノミクス
- ARPU: $5
- 変動費/ユーザー: $0.15
- 粗利/ユーザー: $4.85 (97%)
- LTV/CAC: 初期はCAC極小（オーガニック流入）で成立
