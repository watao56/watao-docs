# 🧩 QuizBio Lite

## 概要
「3問診断」を最短3分で作ってリンクインバイオに設置できるマイクロSaaS。回答タイプ別におすすめ商品/コンテンツへ分岐し、クリック率を改善。

## 海外事例分析
- **主要事例:** Typeform / Interact Quiz / Linktree
- **観察:** 海外のCreator Economyでは静的link-in-bioから診断型導線へ移行。
- **日本ローカライズ仮説:** 日本は「診断コンテンツ」文化が強く、SNSとの相性が高い。

## ターゲット
- 主: 個人クリエイター、コーチ、小規模EC
- 副: LP制作代行者
- JTBD: フォロワーを「最適な1リンク」に誘導したい

## 料金
- Starter $5/月
- 目標は**月$20**なので、初期は3〜5人課金で到達可能な設計にする

## ユーザーフロー
1. テンプレ選択（美容/転職/学習など）
2. 3問を編集して公開
3. bioリンクに貼る
4. 回答分布とCVクリックを確認

## デザインコンセプト
- キーワード: Playful, colorful chips
- 共有したくなるUI: 診断結果カードの共有UI
- MVP時点のUI要素: クイズビルダー、結果テンプレ、分析画面

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
- AI: $0〜$1（質問文提案のみ任意）
- 通知/メール: $0（無料枠）
- 合計: **$0.8〜$2.5**

## MVPスコープ（2週間）
- Must: 3問分岐クイズ作成
- Must: 結果別リンク分岐
- Should: 簡易分析（回答率/CVクリック）
- Won't: 複雑なチーム権限/多言語高度対応

## マーケ計画
- 初動: X/Discordで「作例」を毎日投稿
- ループ: 診断結果カード共有→新規作成者流入
- 獲得目標: 2週間で無料ユーザー30人、転換率10%で有料3人

## 技術スタック
- Next.js 15 / TypeScript / Tailwind / shadcn/ui
- Supabase Postgres / Prisma
- OpenAI gpt-4o-mini（要約・分類） or 画像系はReplicate/FLUX Schnell
- Stripe Payment Links

## リスク
- テンプレ品質が低いと離脱
- スパム利用
- 対策: ユースケース限定 + 無料枠制御 + 監視ダッシュボード

## 競合分析
- 既存競合: Typeform, Interact, Linktree
- 差別化: 「3問診断×bio導線改善」の一点集中で安価

## $20達成シナリオ
- 価格: $5/月
- 必要有料ユーザー: 4人
- 目標到達時期: 1〜2ヶ月目

## ユニットエコノミクス
- ARPU: $5
- 変動費/ユーザー: $0.08
- 粗利/ユーザー: $4.92 (98%)
- LTV/CAC: 初期はCAC極小（オーガニック流入）で成立
