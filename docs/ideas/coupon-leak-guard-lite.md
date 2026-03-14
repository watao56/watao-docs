# 🛡️ CouponLeak Guard Lite

## 概要
🛡️ CouponLeak Guard Liteは、月$20達成を最短で狙う小粒プロダクト。カテゴリは**保険寄りマイクロSaaS（公開クーポン不整合検知）**。保険型偏重を避けるため、攻めの価値（作る/見せる/広げる）を中心に設計した。

## 海外事例分析
- 参照事例: **Visualping / Hexowatch / Shopify Apps**
- 共通点: 「1画面で価値が伝わる」「シェアされやすい成果物」「テンプレ化で継続利用」
- 日本向けローカライズ:
  - 日本語コピーの自然さ（敬体/常体の切り替え）
  - X/Instagram/LINEでの見え方最適化
  - 小規模運用者（個人開発者/1人事業）向け価格

## ターゲット
- 主対象: 個人開発者、クリエイター、小規模EC、副業層
- 課題: 時間不足、発信継続の難しさ、クリエイティブ品質のばらつき

## 料金
- Free: 月10回まで
- Pro: **$6/月**
- Plus: **$12/月**（チーム・追加枠）

## ユーザーフロー
1. 初回オンボーディング（1分）
2. 入力（URL/画像/メモ）
3. AI生成（30〜60秒）
4. 編集・保存
5. 共有（SNS/リンク）

## デザインコンセプト
- コンセプト: **“作業感ではなく作品感”**
- 方向性: 高コントラスト、太めタイポ、余白多め、カードUI
- 共有導線: 生成物に小さな透かし付きURL（自然拡散）

## アーキテクチャ
- Frontend: Next.js + Tailwind + shadcn/ui
- Backend: Next.js Route Handlers / Supabase Edge Functions
- Auth/DB: Supabase
- Storage: Supabase Storage
- Queue: Upstash Redis + QStash（無料枠）
- Hosting: Vercel（Hobby） or AWS Lightsail + Docker

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, type, input_json, status, created_at)
- outputs(id, project_id, asset_url, meta_json, created_at)
- usage_events(id, user_id, kind, tokens_in, tokens_out, cost_usd, created_at)
- subscriptions(id, user_id, plan, stripe_customer_id, status)

## コスト見積もり（月）
- Hosting: $0〜$5（無料枠優先）
- DB/Storage: $0〜$3
- AI API: $2〜$8（mini系/画像は低頻度）
- 合計: **$2〜$12**

## MVPスコープ（2週間）
- 入力→生成→保存→共有の最短ループ
- Stripe決済
- 使用量制限
- 生成履歴

## マーケ計画
- Xで毎日1投稿（生成前/生成後）
- Product Hunt Upcoming登録
- noteで「海外事例→日本向け改善」連載
- 初期10ユーザーへ手動オンボーディング

## 技術スタック
- TypeScript, Next.js, Supabase, Stripe, OpenAI/Replicate, Cloudflare Images(optional)

## リスク
- 競合の機能追随が速い
- 生成品質のブレ
- 無料ユーザー偏重

## 競合分析
- 大手: Canva/Jasper（高機能だが重い）
- 直接競合: 単機能ツール（機能は深いが見た目が弱い）
- 勝ち筋: **日本語特化 + 即シェア + 小額課金**

## $20達成シナリオ
- Pro($6) x 4人 = $24/月
- または Plus($12) x 2人 = $24/月
- 目標達成時の想定粗利: 70〜95%

## ユニットエコノミクス
- ARPU: $6.8
- 変動費/ユーザー: $0.4〜$1.8
- 粗利/ユーザー: $5.0〜$6.4
- 回収期間: 1ヶ月以内（SNSオーガニック獲得）
