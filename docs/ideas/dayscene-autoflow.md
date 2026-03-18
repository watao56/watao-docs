# 🌤️ DayScene Autoflow

## 概要
Rise/Sunsama系の海外潮流を、日本向けに「気分と体力を先に入れる」設計へ再構成。朝30秒チェックインで、当日のタスク配置を自動最適化。

## 海外事例分析
- **Sunsama / Rise / Routine**: カレンダー一体型タスク管理が主流。
- 日本はToDo過多で燃え尽きやすく、**体力パラメータ連動**は未浸透。

## ターゲット
- 副業持ち会社員
- 1人開発者
- 日々の集中波が大きい人

## 料金
- Free: 3プロジェクトまで
- Plus: $7/月（自動再配置/振り返り）
- Pro: $12/月（AI週次レビュー）

## ユーザーフロー
1. 朝に「体力・気分・空き時間」を入力
2. AIがタスクを3ブロックに再配置
3. 実行後に1タップ記録
4. 週次で改善提案

## デザインコンセプト
「天気UI」。晴れ/曇り/雨でその日の負荷を可視化。

## アーキテクチャ
- Next.js + PWA
- Supabase DB
- OpenAI APIでスケジューリング提案
- Google Calendar API連携

## DB設計
- users(id, email, plan)
- daily_checkins(id, user_id, energy, mood, free_minutes, date)
- tasks(id, user_id, title, estimate_min, priority, project)
- schedules(id, user_id, date, block_json)
- weekly_reviews(id, user_id, week_key, summary)

## コスト見積もり（月）
- Supabase: $0〜$25
- OpenAI API: $4
- Hosting(Vercel free): $0
- 合計: 約$4

## MVPスコープ
- チェックイン
- 3ブロック自動配置
- 実績ログ
- 週次サマリ

## マーケ計画
- 「今日は雨モードだから軽タスクだけ」のスクショ訴求
- IndieHackers / Xでテンプレ配布

## 技術スタック
Next.js, TypeScript, Supabase, OpenAI API, Google Calendar API

## リスク
- 習慣化失敗 → 30秒導線に固定
- 提案精度低い → 手動調整を前提UIに

## 競合分析
- Todoist: 体調軸なし
- Sunsama: 高価格帯
- 差別化: **低価格×体力連動×日本語UI**

## $20達成シナリオ
- Plus $7 × 3人 = $21

## ユニットエコノミクス
- ARPU: $7
- 変動費/ユーザー: $0.7
- 粗利: $6.3 (90%)
