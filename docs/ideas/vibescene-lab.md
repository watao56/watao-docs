# 🎨 VibeScene Lab

## 概要
テキスト1行（例:「雨の渋谷、ネオン、シネマ調」）から、**SNS投稿用の統一ビジュアルセット（縦動画カバー/カルーセル/サムネ）**を一括生成するAIデザインSaaS。単発生成ではなく「シリーズ感」を重視し、ブランド一貫性を自動維持する。

- カテゴリ: AI×クリエイティブ
- 目標: 月$20（$8プラン×3人で達成）
- 想定: 個人クリエイター/小規模EC/SNS運用代行

## 海外事例分析
- Headliner: 「1入力→複数配信フォーマット」需要が強い
- Canva Magic Design: テンプレ+AIの量産ワークフローが主流
- Adobe Express: ブランドキット連動が継続率を押し上げる

示唆: 日本市場では「英語UIが重い」「日本語フォント相性」が弱点。**和文タイポ最適化**で差別化可能。

## ターゲット
- 週3本以上SNS投稿する個人
- デザイン外注コストを削りたい副業層
- 1人広報のスモールチーム

## 料金
- Free: 月10セット（透かしあり）
- Starter: $8/月（80セット、透かしなし）
- Pro: $15/月（200セット、ブランドプリセット3つ）

## ユーザーフロー
1. トーン入力（世界観/色/用途）
2. 参照画像1枚アップロード（任意）
3. 6枚セット自動生成
4. フォント/色を1クリック微調整
5. PNG/WEBPで書き出し、投稿

## デザインコンセプト
- 「夜のデザインツール」: 暗色UI+強いアクセント
- 操作は3画面以内
- 生成結果は“並べた瞬間に映える”グリッド重視

## アーキテクチャ
- Next.js (App Router) + Cloudflare R2
- API: FastAPI (Render Free)
- 画像生成: Replicate（Flux Schnell系）
- キュー: Upstash Redis
- 認証: Clerk Free

## DB設計
- users(id, email, plan, credits, created_at)
- projects(id, user_id, prompt, style_preset, created_at)
- assets(id, project_id, type, url, width, height)
- billing_events(id, user_id, event_type, amount, at)

## コスト見積もり（月）
- Hosting: $0〜$5（Vercel/Render無料枠優先）
- DB/Redis: $0（Supabase/Upstash無料枠）
- AI生成: 約$0.015/セット
- 50有料ユーザー時AIコスト: 約$6〜$12

## MVPスコープ
- 3スタイル（Cinematic/Minimal/Pop）
- 3出力比率（1:1, 4:5, 9:16）
- 一括書き出し
- Stripe決済

## マーケ計画
- Xで「ビフォー→アフター」動画を毎日投稿
- 「無料で3セット」導線を固定ポスト化
- Canva/CapCut難民向け比較記事

## 技術スタック
Next.js / FastAPI / Supabase / Upstash / Replicate / Stripe / Cloudflare R2

## リスク
- 画像生成品質のブレ
- 著作権/類似性懸念
- 生成遅延による離脱

## 競合分析
- Canva: 多機能だが深い設定が必要
- Adobe Express: 高機能だが個人には重い
- 本案: **シリーズ一貫性に特化**し操作量を削減

## $20達成シナリオ
- Starter $8 × 3人 = $24 MRR
- もしくは Pro $15 × 2人 = $30 MRR
- 初月目標はX経由の無料登録40→有料転換7.5%

## ユニットエコノミクス
- ARPU: $9.5
- 変動費（AI+配信）: $1.0/user
- 粗利: $8.5/user（粗利率89%）
- 回収期間: 1か月未満（広告ゼロ運用前提）
