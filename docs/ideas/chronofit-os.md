# 🧠 ChronoFit OS

## 概要
睡眠・集中可能時間・予定密度から、**その日のタスク順を自動で再配置**する個人生産性ツール。Sunsama/Routine系の次フェーズとして、エネルギー優先の実行支援に特化。

## 海外事例分析
- Rise等で睡眠可視化の習慣化が進む
- Sunsama/Motionは予定統合が強いが、体調データ活用は限定的
- 日本ではタスク管理は多いが、**体調×実行順最適化**はまだニッチ

## ターゲット
- 副業ワーカー
- フリーランス
- ADHD傾向のある知的労働者

## 料金
- Free: 1プロジェクト、毎日3提案
- Solo: $7/月（無制限提案）
- Pro: $12/月（Googleカレンダー同期）

## ユーザーフロー
1. 朝に体調（睡眠/気分）を入力
2. 今日のタスクを取り込み
3. AIが「今やる順」と見積りを提案
4. 実行ログを記録
5. 夜に翌日の改善フィードバック表示

## デザインコンセプト
- 「ゲームHUD風の1日ダッシュボード」
- 色で負荷を表現（緑=軽作業、赤=重作業）

## アーキテクチャ
- Next.js + Local-first(PWA)
- Supabaseで同期
- ルールベース + 軽量LLMで再配置
- Cronで朝の提案を生成

## DB設計
- users(id, timezone, chronotype)
- tasks(id, user_id, title, effort, deadline)
- daily_state(id, user_id, sleep_hours, mood)
- plans(id, user_id, date, order_json)
- logs(id, task_id, actual_minutes, done)

## コスト見積もり（月）
- Hosting: $5
- Supabase: $0〜$25
- AI提案: $2（miniモデル中心）
- 合計: 約$7〜$32

## MVPスコープ
- 体調入力
- タスク再配置
- 実行ログ
- 翌日フィードバック

## マーケ計画
- 「朝3分で今日が決まる」訴求
- ADHD/副業コミュニティでβ配布
- 7日チャレンジ導線

## 技術スタック
Next.js, TypeScript, Supabase, OpenAI mini, Vercel Cron

## リスク
- データ入力が面倒で離脱
- 提案精度の個人差

## 競合分析
- Sunsama: 手動設計が中心
- Motion: 価格が高め
- 差別化: **体調入力前提の軽量再配置**

## $20達成シナリオ
- Solo 3人で$21
- COGS小さく即黒字

## ユニットエコノミクス
- ARPU: $7
- COGS/user: $0.8
- 粗利: $6.2（88.6%）
