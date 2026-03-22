# 🎨 StreetCast Remix

## 概要
海外で伸びている「テキストから縦動画広告を即量産（CapCut Commerce Pro / Canva Magic Media文脈）」を、日本向けに**“街頭ポスター風モーション”**へ特化したマイクロプロダクト。商品URLか短い説明文を入れると、15秒縦動画と静止画3種を同時出力。

## 海外事例分析
- **CapCut Commerce Pro**: 商品訴求動画の高速量産
- **Canva Magic Media**: 非デザイナー向けのテンプレ生成
- **Adobe Express**: SNS投稿サイズの即時展開
- 日本では「和文タイポ+余白設計+季節感テンプレ」の最適化余地が大きい

## ターゲット
- 小規模D2C
- 個人クリエイター
- 店舗SNS担当（1人運用）

## 料金
- Free: 月10本
- Pro: $8/月（無制限に近い月200本上限）
- Team: $15/月（2シート）

## ユーザーフロー
1. 商品URL/説明を入力
2. トーン選択（渋谷ネオン/ミニマル和風/ポップ）
3. 15秒動画+静止画3枚を生成
4. そのままX/Instagram用に書き出し

## デザインコンセプト
- 「駅前ポスター × SNS」
- 黒背景+高彩度アクセント、太字日本語フォント
- 1画面1メッセージの強い視認性

## アーキテクチャ
- Next.js (App Router) + API Routes
- 生成: OpenAI画像/動画API（低解像度優先）
- 保存: S3
- ジョブ: EventBridge + Lambda
- 認証: Clerk free tier

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, title, tone, source_text, created_at)
- renders(id, project_id, type, status, s3_key, cost_cents, created_at)
- usage_monthly(id, user_id, month, render_count, ai_cost_cents)

## コスト見積もり（月）
- AWS: $3.5（S3/Lambda/CloudFront小規模）
- AI: $6（Pro 5人想定、低コストモデル）
- 合計: $9.5

## MVPスコープ
- 入力→生成→DL
- 3スタイル固定
- Usage上限
- Stripe決済

## マーケ計画
- Xで「同じ商品を3トーン比較」短尺投稿
- 制作事例を毎日1本
- 初月はDM営業（D2C 30件）

## 技術スタック
- Next.js / TypeScript / Tailwind
- PostgreSQL (Supabase)
- Stripe
- AWS S3 + Lambda

## リスク
- 生成品質のブレ
- 既存大手との差別化不足
- API単価上振れ

## 競合分析
- Canva/CapCutは汎用、StreetCast Remixは**日本語広告トーン特化**
- テンプレ深掘りでニッチ優位

## $20達成シナリオ
- Pro 3人 = $24/月 → 目標達成

## ユニットエコノミクス
- Pro ARPU: $8
- 1ユーザー月変動費: $1.2
- 粗利: $6.8（85%）
