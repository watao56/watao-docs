# 👥 SprintMates

## 概要
毎晩20分の“公開作業スプリント”を行う超軽量コミュニティ。AIが進行役となり、開始宣言→作業→成果1行投稿までを回す。海外のbody doubling文化を日本向けに短時間化。

## 海外事例分析
- Focusmate: 1on1作業同席
- Caveday: 集団集中セッション
- Discord study-with-me文化: 無料だが継続率が課題
- 日本ギャップ: 短時間・匿名運用に最適化された有料版が少ない

## ターゲット
- 副業開発者
- 資格勉強勢
- 在宅ワーカー

## 料金
- Free: 週2回参加
- Plus: $5/月（毎日参加、履歴分析）
- Crew: $15/月（小グループ固定席）

## ユーザーフロー
1. 今日の目標を15文字で宣言
2. 20分タイマー開始
3. 終了時に成果1行投稿
4. AIが継続率・得意時間帯を可視化

## デザインコンセプト
- 「夜の作業部屋」
- 暖色グラデ、ローファイUI、達成バッジ

## アーキテクチャ
- Next.js + Supabase Realtime
- スケジュール通知: cron + Discord Webhook
- AI分析: gpt-4o-mini バッチ

## DB設計
- users(id, name, plan)
- sprints(id, starts_at, duration_min, room_id)
- sprint_logs(id, sprint_id, user_id, goal, result)
- streaks(id, user_id, current_days, best_days)
- events(id, user_id, kind, payload)

## コスト見積もり（月）
- Supabase: $0〜$25（初期$0）
- Vercel: $0
- AI要約: $2
- 合計: **約$2〜$5**

## MVPスコープ
- 1部屋運用
- 毎日固定2スロット
- 成果ログと連続記録

## マーケ計画
- 「#20分だけやる」投稿テンプレ配布
- Discordコミュニティ提携
- 7日連続達成者のUGC再投稿

## 技術スタック
Next.js / Supabase / Discord API / cron / OpenAI mini

## リスク
- 初期アクティブ不足で部屋が過疎化
- モデレーション負荷

## 競合分析
- Focusmateより短時間・匿名重視
- Discord無料鯖より継続UXを商品化

## $20達成シナリオ
- Plus 4人で $20 MRR

## ユニットエコノミクス
- ARPU: $5
- 変動費/人: $0.2
- 粗利率: 96%
