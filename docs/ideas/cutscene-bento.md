# 🎞️ CutScene Bento

## 概要
CutScene Bentoは、AI×クリエイティブ（動画→縦カルーセル）に特化した月$20到達向けマイクロプロダクト。初期は1人開発（AIエージェント実装）で、AWS無料枠+低コスト構成を前提に収益化する。

## 海外事例分析
- 参照: **Captions / OpusClip / CapCut Templates**
- 海外で成立している理由: 短時間で価値が出る / SNS共有しやすい / テンプレ化しやすい
- 日本ローカライズ余地: 日本語フォント最適化、LINE/Discord共有導線、価格を$4〜$9帯に圧縮

## ターゲット
- 主: 個人開発者・副業クリエイター・小規模チーム（1〜5人）
- 副: SNS運用担当、フリーランス

## 料金
- プラン: Free / Solo $6 / Pro $12
- 目標は**3〜5課金ユーザーで月$20突破**

## ユーザーフロー
1. サインアップ（Google or メールリンク）
2. 入力（URL/メモ/テーマ）
3. AIまたはルール処理を実行
4. 結果を保存・共有
5. 再訪を促すリマインド（週次）

## デザインコンセプト
- キーワード: **Playful Utility**（実用的だが見せたくなる）
- UI: bentoグリッド、濃色背景+アクセント1色、3クリック以内完了
- モバイル優先（縦長カード中心）

## アーキテクチャ
- Front: Next.js 15 (App Router) + Tailwind + shadcn/ui
- API: Next.js Route Handlers
- Worker: AWS Lambda（定期処理）
- Queue: SQS（必要時）
- DB: Supabase Postgres
- Auth: Supabase Auth
- Analytics: Plausible self-host or PostHog free

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, title, input_json, output_json, status, created_at)
- runs(id, project_id, token_in, token_out, cost_usd, latency_ms, created_at)
- subscriptions(id, user_id, stripe_customer_id, plan, status, renew_at)
- events(id, user_id, type, meta_json, created_at)

## コスト見積もり（月次）
- Infra: Vercel/Supabase無料枠 + Lambda無料枠中心
- 可変費: AI推論（必要プロダクトのみ）
- 想定: **$3.2/月（10有料ユーザー時）**
- 粗利率目安: 80%〜97%

## MVPスコープ（2週間）
- must: 認証 / 1つの主機能 / 結果保存 / 課金導線（Stripe Checkout）
- should: テンプレート3種 / 共有リンク
- won’t: 高度な権限管理 / ネイティブアプリ

## マーケ計画
- 0→1: Xで「作例」を毎日投稿（14日）
- 1→10: Product Hunt / Reddit / Discordコミュニティ投稿
- 10→: テンプレ配布でSEO流入（比較記事）

## 技術スタック
- TypeScript, Next.js, Supabase, Stripe, AWS Lambda, GitHub Actions
- AI: gpt-4o-mini級 or OSS（必要最低限）

## リスク
- 競合の模倣速度が速い
- 流入チャネル依存（Xアルゴリズム変動）
- AI品質のぶれ（プロンプト管理で緩和）

## 競合分析
- 大手: 多機能だが高価格（$15〜$49）
- 本案: 1ジョブ特化+低価格+日本語UXで差別化

## $20達成シナリオ
- 例1: $5プラン×4人 = $20
- 例2: $7プラン×3人 = $21
- 初月目標: 2人課金、2ヶ月目で4人

## ユニットエコノミクス
- ARPU: $5〜$9
- 変動原価: $0.1〜$1.5 / user / month
- 粗利: $4〜$8 / user / month
- 回収期間: 初月（広告費ゼロ前提）

---

## この案のカテゴリ適合メモ
- カテゴリ: AI×クリエイティブ（動画→縦カルーセル）
- 保険型比率: 5案中1案のみ（Renewal Nudger Lite）
- 過去案との差分: 「監視」ではなく**創作体験/参加体験/行動変容**を主価値に置く
