# 📸 SnapFit Mirror

## 概要
自撮り1枚から「雑誌風ルックカード」を自動生成し、Instagram/LINE向けに縦長テンプレートで即共有できる。日本語フォント最適化と季節タグ自動付与で、個人クリエイター〜小規模アパレルが使いやすい。

## 海外事例分析
- **主要事例:** Lensa / Combyne / Pinterest Collages
- **観察:** 海外では「AIアバター」から「実写ベースのスタイル提案」へ重心が移動。
- **日本ローカライズ仮説:** 日本は「盛りすぎNG」志向が強く、実写保持+軽補正が刺さる。

## ターゲット
- 主: Instagram運用中の個人/小規模アパレル
- 副: 古着店・美容師の私服投稿運用者
- JTBD: 撮影編集の手間を減らしつつ、投稿の見栄えを上げたい

## 料金
- Free, Pro $6/月
- 目標は**月$20**なので、初期は3〜5人課金で到達可能な設計にする

## ユーザーフロー
1. 自撮りをアップロード
2. テイスト（モード/カジュアル等）選択
3. AIが3案のルックカードを生成
4. SNSサイズでDL・投稿

## デザインコンセプト
- キーワード: Magazine, clean, soft neon
- 共有したくなるUI: 「今日の1枚」をショーケース表示
- MVP時点のUI要素: カード生成画面、テンプレ選択、シェアCTA

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
- AI: $3〜$8（画像生成は無料枠優先、超過時のみ課金）
- 通知/メール: $0（無料枠）
- 合計: **$4〜$12**

## MVPスコープ（2週間）
- Must: 画像アップロード+テンプレ3種生成
- Must: 日本語フォント自動最適化
- Should: ハッシュタグ自動提案
- Won't: 複雑なチーム権限/多言語高度対応

## マーケ計画
- 初動: X/Discordで「作例」を毎日投稿
- ループ: 生成カードに薄いウォーターマーク（Powered by SnapFit）で自然流入
- 獲得目標: 2週間で無料ユーザー30人、転換率10%で有料3人

## 技術スタック
- Next.js 15 / TypeScript / Tailwind / shadcn/ui
- Supabase Postgres / Prisma
- OpenAI gpt-4o-mini（要約・分類） or 画像系はReplicate/FLUX Schnell
- Stripe Payment Links

## リスク
- 肖像権/著作権に配慮が必要
- 生成品質のブレ
- 対策: ユースケース限定 + 無料枠制御 + 監視ダッシュボード

## 競合分析
- 既存競合: Canva, Lensa, BeautyPlus
- 差別化: 「投稿前提」の縦長テンプレと日本語タイポ最適化に特化

## $20達成シナリオ
- 価格: $6/月
- 必要有料ユーザー: 4人
- 目標到達時期: 1〜2ヶ月目

## ユニットエコノミクス
- ARPU: $6
- 変動費/ユーザー: $0.7
- 粗利/ユーザー: $5.3 (88%)
- LTV/CAC: 初期はCAC極小（オーガニック流入）で成立
