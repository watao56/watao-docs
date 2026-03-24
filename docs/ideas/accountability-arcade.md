# 🕹️ Accountability Arcade

## 概要
「作業開始ボタン」を押すと2〜4人の即席ルームに入り、25分集中でポイントを獲得する**ゲーム化ボディダブル**。海外のco-working文化を日本向けに短時間×匿名で最適化。

## 海外事例分析
- Focusmate: 1on1の集中セッションが定着。
- Flown系サービス: コミュニティ価値が継続率を押し上げる。
- 日本市場は「匿名・短時間・低価格」の組合せ余地あり。

## ターゲット
- 在宅ワーカー
- 受験/資格勉強層
- 副業クリエイター

## 料金
- Free: 週5セッション
- Pro: **$4/月**（無制限、成績カード）

## ユーザーフロー
1. ワンクリック入室
2. 目標を10秒で宣言
3. 25分集中（カメラ任意、音声なし）
4. 終了後に達成チェック
5. 週次レポート共有

## デザインコンセプト
- レトロゲームUI（ピクセル風）
- 実績バッジ、連続達成エフェクト

## アーキテクチャ
- Front: Next.js
- Realtime: Supabase Realtime
- Matching: Cloudflare Worker Queue
- Optional video: Daily.co (無料枠)

## DB設計
- users(id, plan, timezone)
- sessions(id, room_code, started_at, ended_at)
- session_members(id, session_id, user_id, goal, done)
- streaks(user_id, current_days, best_days)

## コスト見積もり（月）
- Hosting/DB: $0〜$5
- Realtime: $0
- Video API: $0（初期）
- 合計: **$5以下**

## MVPスコープ
- 25分固定ルーム
- 簡易マッチング
- 連続記録
- Stripe課金

## マーケ計画
- Study系Discordと相互連携
- 「今夜3本集中」イベント配信
- Xで週間ランキング投稿

## 技術スタック
Next.js / Supabase / Cloudflare Workers / Stripe

## リスク
- 初期は同時接続不足で体験劣化
- モデレーションコスト

## 競合分析
- Focusmate: 1on1前提
- 本案: **短時間・複数人・ゲーム化**

## $20達成シナリオ
- Pro $4 × 5人 = $20

## ユニットエコノミクス
- ARPPU: $4
- 変動費: $0.5〜$1
- 粗利率: 75%+
