# 📰 ZineFold

## 概要
メモ・画像・URLを投げると、AIが**雑誌風の8ページミニジン**を自動生成。A4印刷/PDF/SNSカルーセルを同時出力し、「作って終わり」でなく**見せたくなる成果物**に変換する。

## 海外事例分析
- Flipsnack: デジタル冊子作成の需要は高いが、日本語デザイン最適化が弱い
- Canva Docs/Magic Design: 生成は強いが「ジン文化」特化ではない
- Substack系ニュースレター: 文章は強いがビジュアル再編集が手間
→ **日本語タイポ・余白設計込みで“ワンクリックジン化”**が差別化。

## ターゲット
- X/Instagramで発信する個人クリエイター
- 勉強記録/読書記録をビジュアル化したい人
- 週報を“作品化”したい小規模チーム

## 料金
- Free: 月2冊
- Creator: $6/月（30冊、ブランド保存）
- Print+: $12/月（印刷プリセット、共同編集）

## ユーザーフロー
1. 素材投入（テキスト/画像/URL）
2. テーマ選択（Brutalist, Minimal, Retro等）
3. AIレイアウト生成（3案）
4. 編集→PDF/SNS出力→共有

## デザインコンセプト
「**Indie雑誌の余白感**」。白背景＋強い見出し＋紙ノイズ。生成物をそのまま投稿できるビジュアル強度を優先。

## アーキテクチャ
- Next.js (App Router)
- API: Cloudflare Workers
- 生成: OpenAI Responses API (mini) + html-to-image
- 保存: Supabase (Postgres + Storage)
- Queue: Cloudflare Queues

## DB設計
- users(id, plan, credits, created_at)
- zines(id, user_id, title, theme, status, pages_json, cover_url)
- assets(id, user_id, kind, src, meta_json)
- exports(id, zine_id, type, url, created_at)
- billing_events(id, user_id, provider, event_type, payload_json)

## コスト見積もり（月）
- Supabase無料枠〜$0
- Cloudflare無料枠〜$5
- AI生成: 1冊あたり$0.01〜$0.03、100冊で$3
- 合計: **$3〜$8**

## MVPスコープ
- 3テーマ固定
- 8ページ固定レイアウト
- PDF出力とSNS画像出力
- Stripe課金

## マーケ計画
- 「今日のジン」テンプレをX配布
- Notionクリエイター向け連携記事
- 7日間ジン化チャレンジをDiscordで開催

## 技術スタック
Next.js / TypeScript / Supabase / Cloudflare / Stripe / OpenAI mini

## リスク
- 著作権素材の混入 → アップロード時チェック + 利用規約
- 生成品質のばらつき → レイアウトルールを先に固定

## 競合分析
Canvaは汎用、Flipsnackは制作導線が重い。ZineFoldは**短時間で作品品質**に寄せる一点突破。

## $20達成シナリオ
Creator($6)を4人獲得で$24達成（最短1〜2か月）。

## ユニットエコノミクス
- ARPU: $6
- 変動費: $0.5/人未満
- 粗利: 約91%
- 回収期間: 初月
