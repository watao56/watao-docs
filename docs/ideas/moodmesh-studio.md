# 🎨 MoodMesh Studio

## 概要
MoodMesh Studioは、月$20達成を狙う小粒SaaS。**AI×クリエイティブ（ムードボード自動生成）**として、初期は1人開発+AIエージェント実装で2週間MVPを前提。

## 海外事例分析
- 参照: **Canva Magic Design、Krea、Milanote**
- 海外トレンド: 「テンプレ配布」より**成果物をすぐ公開できる体験**が伸びている
- 日本向け差分: 日本語フォント最適化、X/Instagram向け比率、LINE共有導線

## ターゲット
- 主: 個人クリエイター / 個人開発者 / 小規模チーム（1〜10名）
- 副: 週1以上で発信・制作・改善を行う人

## 料金
- Free / Solo $7 / Pro $12
- 決済: Stripe（月額）

## ユーザーフロー
1. サインアップ（Google）
2. 入力（テキスト/URL/音声/契約日など）
3. 生成 or チェック実行
4. 出力を編集
5. 共有・エクスポート

## デザインコンセプト
- 「1画面で気持ちよく終わる」
- グラデーション＋大きいCTA＋カードUI
- SNSに貼って映えるサムネを自動生成

## アーキテクチャ
- Frontend: Next.js 15 (App Router) + Tailwind
- Backend: Next.js Route Handlers + Supabase(Postgres/Auth/Storage)
- Queue: Upstash Redis
- Batch: GitHub Actions cron or Supabase Scheduled Functions
- Notification: Discord Webhook / Email (Resend)

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, title, meta_json, created_at)
- jobs(id, project_id, status, input_json, output_json, cost_usd)
- subscriptions(id, user_id, stripe_customer_id, plan, status)
- events(id, user_id, event_type, payload_json, created_at)

## コスト見積もり（月）
- Vercel Hobby: $0
- Supabase Free: $0
- Upstash: $0〜$1
- AI API: $2〜$8（GPT-4o-mini/画像は必要時のみ）
- 合計: **$2〜$9**

## MVPスコープ
- 認証・課金
- コア機能1本
- 共有URL発行
- 基本分析（生成回数、継続率）

## マーケ計画
- Day1: Xで開発ログ投稿
- Day3: Product Hunt upcoming / Zenn記事
- Day7: 先着10人にLifetimeディスカウント
- Day14: 利用事例をカード化して再配信

## 技術スタック
- TypeScript / Next.js / Supabase / Stripe / Upstash
- 監視: Sentry Free
- 分析: PostHog Free

## リスク
- AI出力品質のブレ
- 初期トラフィック不足
- 課金導線の離脱

## 競合分析
- 海外は高機能・英語中心、日本語最適化が弱い
- 本案は**日本語UX + 低価格 + 速い初回体験**で差別化

## $20達成シナリオ
- 例: 月額$6プラン 4人 = $24 MRR
- または 月額$10プラン 2人 = $20 MRR
- 目標期間: リリース後30日以内

## ユニットエコノミクス
- ARPU: $7.5想定
- 変動費: $0.6/ユーザー
- 粗利: $6.9/ユーザー（粗利率92%）
- 回収期間: 初月
