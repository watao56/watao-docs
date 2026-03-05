# 🎬 NeonReel Reply

## 概要
SNSのコメント1行から、**ネオン調の縦動画リプライ（6〜10秒）**を自動生成するマイクロSaaS。X/Instagram/TikTok運用者向けに「返信の見栄え」を最短30秒で作る。海外の“faceless short-form creator tools”潮流を日本語UIで再設計。

## 海外事例分析
- **CapCut Templates**: テンプレ主導の大量生成UXが強い
- **Captions / VEED**: AI字幕・自動演出が中小クリエイターに浸透
- **Canva Magic Media**: 直感的プリセットが継続率を押し上げ
- 日本ギャップ: 「返信専用」「短尺リアクション特化」が薄い

## ターゲット
- 毎日投稿する個人クリエイター（フォロワー1k〜50k）
- 小規模D2CのSNS担当
- 配信者/ポッドキャスター

## 料金
- Free: 月10本（透かしあり）
- Pro: **$5/月**（120本、透かしなし、ブランドカラー保存）
- Solo Lifetime: $19（初期検証用）

## ユーザーフロー
1. コメント文を貼り付け
2. テンポ（calm/energetic）とスタイル選択
3. 30秒で縦動画生成
4. ダウンロード→投稿

## デザインコンセプト
- 「夜のネオン看板」風（黒背景＋発光グラデ）
- 1画面1アクション、迷わない導線
- 生成結果を“見せたくなる”アニメで演出

## アーキテクチャ
- Frontend: Next.js + Tailwind
- Backend: Next.js Route Handlers
- AI: OpenAI mini系（コピー整形）+ FFmpegテンプレ合成
- Storage: Cloudflare R2（低コスト）
- Auth/Billing: Clerk + Stripe
- Hosting: Vercel Hobby

## DB設計
- users(id, email, plan, created_at)
- projects(id, user_id, input_text, style, status, created_at)
- renders(id, project_id, seconds, storage_url, cost_usd, created_at)
- subscriptions(id, user_id, stripe_sub_id, plan, renew_at)

## コスト見積もり（月）
- Vercel/R2/DB: $8
- AI整形: $2
- 合計: **$10**（100〜150本生成規模）

## MVPスコープ
- 3テンプレ（Neon Pulse / Retro Grid / Mono Flash）
- 日本語最適化字幕
- Stripe決済
- ダウンロード履歴

## マーケ計画
- Xで「1コメント→動画化」Before/Afterを毎日投稿
- ハッシュタグ: #動画編集 #個人開発 #運用代行
- 初期20人に無料クレジット配布→UGC誘発

## 技術スタック
Next.js, TypeScript, Tailwind, PostgreSQL(Supabase), FFmpeg, OpenAI API, Stripe

## リスク
- テンプレ飽き: 週1で新演出追加
- 生成待ち時間: 非同期キュー化で改善
- 著作権: 音源はロイヤリティフリーのみ

## 競合分析
- CapCut: 高機能だが返信特化ではない
- Canva: 汎用強いが短尺反応UXが弱い
- 差別化: **「コメント返信専用」「30秒以内」「日本語口語最適化」**

## $20達成シナリオ
- Pro($5)×4ユーザー = $20
- 想定CV: Free 40人中10%課金で達成

## ユニットエコノミクス
- ARPU: $5
- 変動費/人: $0.8
- 粗利/人: $4.2（84%）
- 回収: 初月で黒字化可能
