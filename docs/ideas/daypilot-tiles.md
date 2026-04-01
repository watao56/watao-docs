# 🧩 DayPilot Tiles

## 概要
**DayPilot Tiles**は、今日のタスクを「8枚のタイル」に自動再配置し、次にやる1つだけを明確化するパーソナル生産性ツール。  
ToDoを増やすのではなく、**迷いコストを減らす**ことに特化。

## 海外事例分析
- **Sunsama**: 1日の計画体験を商品化
- **Goblin Tools**: 認知負荷を下げる分解支援が支持
- **Motion**: 自動スケジュール最適化の需要
- 示唆: 日本市場は高機能より「迷わないUI」が有効

## ターゲット
- タスク過多のフリーランス
- 仕事/学習の並行ユーザー
- ADHD傾向の自己管理ユーザー

## 料金
- Free: 日次3回リプラン
- Pro: **$5/month**（無制限＋週報）

## ユーザーフロー
1. タスクを箇条書き入力
2. AIが8タイルへ整理（今すぐ/後で/捨てる）
3. 1タップで25分開始
4. 終了後に次タイルを提案

## デザインコンセプト
- Notionより軽く、ゲームHUDより静か
- 角丸タイル＋ミニアニメーション
- 「次これ」表示を常時固定

## アーキテクチャ
- Front: Next.js
- Backend: Cloudflare Workers
- DB: D1 or Supabase
- AI: OpenAI miniモデル（短文分類のみ）

## DB設計
- users(id, email, plan)
- tasks(id, user_id, content, priority, state)
- tile_sessions(id, user_id, created_at, focus_count)
- focus_logs(id, session_id, task_id, minutes)

## コスト見積もり（月）
- Cloudflare: $5
- DB: $5
- AI: $4
- 合計: **$14**

## MVPスコープ
- タスク分類
- 8タイルUI
- 25分タイマー
- 週報メール

## マーケ計画
- 「ToDo地獄→8タイル」比較投稿
- Product Hunt mini版ローンチ
- ADHDコミュニティ向け体験会

## 技術スタック
Next.js / Cloudflare Workers / D1 / Stripe / Resend

## リスク
- AI分類精度
- 習慣化失敗
- 既存ToDoツールとの競合

## 競合分析
- Todoist: 管理は強いが“次の1手”が弱い
- Sunsama: 高価格帯
- 本案: **$5で迷い解消**に特化

## $20達成シナリオ
- Pro 4人 = $20/月

## ユニットエコノミクス
- ARPU: $5
- 変動費: 1人あたり$0.6程度
- 粗利率: 約88%
