# 🧾 FocusReceipt

## 概要
ブラウザ/PC作業ログを1日1枚の「作業レシート」に変換し、何に時間を使ったかを可視化する個人向けツール。AIが無駄時間を短い改善提案に変換。

## 海外事例分析
- **主要事例:** Rize / RescueTime / Reflect
- **観察:** 海外のタイムトラッカーはデータ過多で継続率が落ちる傾向。
- **日本ローカライズ仮説:** 日本では「家計簿のように1枚で振り返れる」表現が受ける。

## ターゲット
- 主: 在宅ワーカー、個人開発者
- 副: 時間管理に悩む学生/副業層
- JTBD: 詳細ログではなく今日の使い方をざっくり把握したい

## 料金
- Free, Plus $4/月
- 目標は**月$20**なので、初期は3〜5人課金で到達可能な設計にする

## ユーザーフロー
1. Chrome拡張を導入
2. 日次でアクティブ時間を自動収集
3. AIが作業レシート画像を生成
4. 翌日の改善アクション1つを提案

## デザインコンセプト
- キーワード: Receipt, mono, calm
- 共有したくなるUI: 家計簿風UIで直感理解
- MVP時点のUI要素: 日次カード、週比較グラフ、改善チップ

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
- AI: $0.5〜$2（短文提案のみ）
- 通知/メール: $0（無料枠）
- 合計: **$1〜$3.5**

## MVPスコープ（2週間）
- Must: 拡張でURLカテゴリ別集計
- Must: 日次カード生成
- Should: 改善提案1行
- Won't: 複雑なチーム権限/多言語高度対応

## マーケ計画
- 初動: X/Discordで「作例」を毎日投稿
- ループ: レシート画像をSNS共有→同じ悩み層が流入
- 獲得目標: 2週間で無料ユーザー30人、転換率10%で有料3人

## 技術スタック
- Next.js 15 / TypeScript / Tailwind / shadcn/ui
- Supabase Postgres / Prisma
- OpenAI gpt-4o-mini（要約・分類） or 画像系はReplicate/FLUX Schnell
- Stripe Payment Links

## リスク
- ブラウザ権限への不安
- ログ精度差
- 対策: ユースケース限定 + 無料枠制御 + 監視ダッシュボード

## 競合分析
- 既存競合: RescueTime, Rize, Timing
- 差別化: 「分析」ではなく「1枚の見える化」に極振り

## $20達成シナリオ
- 価格: $4/月
- 必要有料ユーザー: 5人
- 目標到達時期: 1〜2ヶ月目

## ユニットエコノミクス
- ARPU: $4
- 変動費/ユーザー: $0.16
- 粗利/ユーザー: $3.84 (96%)
- LTV/CAC: 初期はCAC極小（オーガニック流入）で成立
