# 📢 AdPolicy Scout Lite

## 概要
Meta/Google/TikTok広告ポリシー更新を毎日差分要約し、日本語で「あなたの業種に関係あるか」を判定して通知する軽量ツール。保険型はこの1案のみ。

## 海外事例分析
- **主要事例:** TermsFeed / Meta Ads Policy updates
- **観察:** 海外では広告アカウント停止リスク回避ニーズが継続的に強い。
- **日本ローカライズ仮説:** 日本の中小事業者は英語一次情報の追跡が難しい。

## ターゲット
- 主: 広告運用代行1〜3名チーム
- 副: D2C小規模事業者
- JTBD: 規約変更を見落として配信停止になるリスクを減らしたい

## 料金
- Solo $7/月
- 目標は**月$20**なので、初期は3〜5人課金で到達可能な設計にする

## ユーザーフロー
1. 業種と利用媒体を設定
2. 毎日ポリシー差分を収集
3. AIが関連度判定して通知
4. 対応ToDoをチェックリスト化

## デザインコンセプト
- キーワード: Radar, alert-lite, clarity
- 共有したくなるUI: 「影響度バッジ（高/中/低）」を色で表示
- MVP時点のUI要素: 媒体別タイムライン、差分ビュー、ToDo

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
- AI: $2〜$4（差分要約と関連度判定）
- 通知/メール: $0（無料枠）
- 合計: **$3〜$7**

## MVPスコープ（2週間）
- Must: 媒体3種の差分クロール
- Must: 関連度判定通知
- Should: 対応チェックリスト
- Won't: 複雑なチーム権限/多言語高度対応

## マーケ計画
- 初動: X/Discordで「作例」を毎日投稿
- ループ: 運用者コミュニティで週次まとめを無料公開→有料誘導
- 獲得目標: 2週間で無料ユーザー30人、転換率10%で有料3人

## 技術スタック
- Next.js 15 / TypeScript / Tailwind / shadcn/ui
- Supabase Postgres / Prisma
- OpenAI gpt-4o-mini（要約・分類） or 画像系はReplicate/FLUX Schnell
- Stripe Payment Links

## リスク
- 一次情報の取得仕様変更
- 通知過多で疲れる
- 対策: ユースケース限定 + 無料枠制御 + 監視ダッシュボード

## 競合分析
- 既存競合: Termly, 手動ウォッチ, agency内製
- 差別化: 広告規約に特化し「影響度×行動提案」まで出す

## $20達成シナリオ
- 価格: $7/月
- 必要有料ユーザー: 3人
- 目標到達時期: 1〜2ヶ月目

## ユニットエコノミクス
- ARPU: $7
- 変動費/ユーザー: $0.63
- 粗利/ユーザー: $6.37 (91%)
- LTV/CAC: 初期はCAC極小（オーガニック流入）で成立
