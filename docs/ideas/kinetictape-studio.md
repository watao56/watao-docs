# 🎬 KineticTape Studio

## 概要
テキスト/音声メモを、**90年代VHS風の縦動画ポスター**に自動変換してSNS投稿まで一気通貫するAIクリエイティブツール。海外の「rawでエモい表現」トレンドを日本語UIで最短化する。

## 海外事例分析
- **Captions / CapCut Templates**: テンプレ起点の高速制作が定着
- **Lapse系のノー加工・質感重視トレンド**: polishedよりauthenticが刺さる
- **Canva Magic Design**: 「素材→即使える1枚」に価値集中
- 日本では「VHS/lo-fi縦動画」特化SaaSはまだ薄い

## ターゲット
- X/Instagram/TikTokで日次発信する個人開発者・デザイナー
- 1人広報の小規模SaaS運営者

## 料金
- Free: 月10本（透かしあり）
- Pro: $6/月（透かしなし・ブランドプリセット）
- Goal: 4人で$24達成

## ユーザーフロー
1. メモ貼り付け or 音声アップロード
2. テーマ選択（Retro/Neon/Mono）
3. AIが3案生成
4. 1クリックで縦動画書き出し+予約投稿文生成

## デザインコンセプト
- 「古いのに新しい」: 粒状感/テープノイズ/大胆タイポ
- 3タップ完結、編集UIは極小

## アーキテクチャ
- Next.js (App Router) + Cloudflare R2
- 生成: Replicate(動画) + OpenAI mini(コピー)
- 非同期処理: AWS Lambda + SQS
- 認証: Clerk

## DB設計
- users(id, plan, credits)
- projects(id, user_id, source_type, status)
- renders(id, project_id, preset, output_url, cost_cents)
- publish_drafts(id, render_id, platform, caption)

## コスト見積もり（月）
- Vercel/CF無料枠: $0
- Replicate: $6（100〜150本想定）
- OpenAI mini: $1
- 合計: 約$7

## MVPスコープ
- テキスト→動画生成
- 3プリセット
- 書き出し/履歴
- Stripe課金

## マーケ計画
- 「#今日の進捗」投稿テンプレを無料配布
- Product Hunt mini launch
- 日本語の作例ギャラリー運用

## 技術スタック
Next.js / TypeScript / Supabase Postgres / Replicate / OpenAI / Stripe

## リスク
- 生成コスト増: 生成回数上限で制御
- 著作権懸念: 素材はユーザー入力のみ利用

## 競合分析
- Canva/CapCutは汎用で操作が重い
- 本サービスは「進捗投稿専用」で速度特化

## $20達成シナリオ
- Xで作例30本投稿→無料20人→CVR20%でPro4人

## ユニットエコノミクス
- ARPU: $6
- 変動費/人: $1.2
- 粗利/人: $4.8（粗利率80%）
