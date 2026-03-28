# 🛋️ ShelfScene AR

## 概要
EC商品画像1枚から、**海外D2Cで流行中の「生活シーン合成＋短尺ループ動画」**を自動生成するマイクロSaaS。Instagram/TikTok投稿用の静止画3枚＋8秒ループをワンクリックで出力。

## 海外事例分析
- **Photoroom**: 背景生成の大量利用が進むが、動画化は限定的
- **Pebblely**: 商品写真の文脈背景生成が浸透
- **Arcads/Creatify系**: AI広告動画は伸びるが、小規模ECには重い
- 日本ギャップ: 「写真は撮るが、見せ方の動き演出まで手が回らない」層が多い

## ターゲット
- Shopify/BASEの小規模D2C
- ハンドメイド作家
- Instagram運用代行の個人

## 料金
- Free: 月10生成（透かしあり）
- Starter: **$9/mo**（120生成）
- One-shot: $5（50生成パック）

## ユーザーフロー
1. 商品画像アップロード
2. テーマ（Nordic / Street / Minimal）選択
3. AIが背景・光源・小物を合成
4. 画像3枚＋8秒ループMP4を書き出し
5. SNSへ投稿

## デザインコンセプト
"**Gallery-grade in 30 sec**"。余白大きめ、白黒基調＋アクセント1色、生成結果をカードUIで見せる。

## アーキテクチャ
- Front: Next.js (Vercel)
- API: AWS Lambda + API Gateway
- Queue: SQS
- Storage: S3
- AI: Replicate (image-to-image + short video model)
- Auth/Billing: Clerk + Stripe

## DB設計
- users(id, email, plan, credits, created_at)
- projects(id, user_id, source_image_url, theme, status)
- assets(id, project_id, type, url, token_cost)
- usage_logs(id, user_id, model, cost_usd, created_at)
- subscriptions(id, user_id, stripe_sub_id, status)

## コスト見積もり（月）
- Vercel Hobby: $0
- AWS (S3/Lambda/SQS): $4
- AI推論: $12（有料3〜5人想定）
- 合計: **$16**

## MVPスコープ
- 商品1枚→画像3枚生成
- 8秒ループ動画1種
- 3テーマ固定
- Stripe課金

## マーケ計画
- Xで「1枚写真→映える棚動画」毎日投稿
- Etsy/BASEコミュニティに無料枠配布
- 制作代行向け紹介制度（1契約$10）

## 技術スタック
Next.js / TypeScript / Tailwind / Lambda / S3 / SQS / PostgreSQL(Supabase)

## リスク
- 生成品質のブレ
- モデル単価上昇
- 既存ツールの機能追加競争

## 競合分析
- Photoroom: 静止画強いが日本語導線と小規模運用導線が弱い
- Canva: 汎用的で、EC特化テンプレ深度が浅い

## $20達成シナリオ
- Starter 3ユーザー = $27 MRR
- 変動費約$9、粗利$18 + One-shot 1件で**$20超**

## ユニットエコノミクス
- ARPU: $9
- 変動費/有料ユーザー: $2.8
- 粗利/有料ユーザー: $6.2
- 回収期間: <1ヶ月
