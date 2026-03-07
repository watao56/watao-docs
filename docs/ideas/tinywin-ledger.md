# 🏁 TinyWin Ledger

## 概要
Git/カレンダー/ToDoから「今日の小さな達成」を自動抽出し、1分で振り返れる**勝ちログ生産性ツール**。海外で伸びる"small wins"メンタルモデルを日本語実務に適用。

## 海外事例分析
- **Reflect / Stoic**: 日次振り返り
- **WakaTime weekly reports**: 自動記録の継続性
- **Bullet Journal community**: 小達成の可視化価値

## ターゲット
- リモートワーカー
- 継続が苦手な学習者
- 個人開発者

## 料金
- Free: 連携1つ
- Plus: $5/月
- Pro: $9/月（チーム2名まで）

## ユーザーフロー
1. GitHub/Google Calendar連携
2. 毎日20時にAIが"Tiny Wins 3件"を提案
3. ユーザーが採用/修正
4. 週次に"Win Heatmap"を生成

## デザインコンセプト
- "Quiet Confidence"
- 暗色背景+蛍光ラインで達成を強調
- 余計な通知を極小化

## アーキテクチャ
- Front: SvelteKit
- Backend: Supabase Edge Functions
- Integrations: GitHub API / Google Calendar API
- AI: OpenAI mini
- Scheduler: cron (OpenClaw or Supabase)

## DB設計
- users(id, email, tz, plan)
- integrations(id, user_id, kind, token_ref)
- win_candidates(id, user_id, date, text, source)
- wins(id, user_id, date, text, score)
- weekly_cards(id, user_id, week, image_url)

## コスト見積もり（月）
- Supabase: $0
- Vercel: $0
- AI API: $3
- Storage/CDN: $1
- 合計: **$4〜8**

## MVPスコープ
- GitHub連携
- 毎日3件提案
- 週次ヒートマップPNG

## マーケ計画
- "今日の3勝"テンプレをX配布
- 開発者向けニュースレターで紹介
- Product Huntの"build in public"タグ狙い

## 技術スタック
SvelteKit / Supabase / OpenAI mini / GitHub API

## リスク
- 連携設定離脱
- 提案精度が低いと継続率悪化

## 競合分析
- 日記アプリ: 手入力前提で面倒
- タスク管理: 完了ログの情緒価値が弱い
- 差別化: 自動抽出×感情設計

## $20達成シナリオ
- Plus($5)×4人 = $20

## ユニットエコノミクス
- ARPU: $5.8
- COGS/user: $0.5
- 粗利: 91%
