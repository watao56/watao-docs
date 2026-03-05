# 🗺️ Questline Solo

## 概要
1日のタスクを“クエスト化”し、完了するとマップが埋まっていく**視覚重視の個人生産性ツール**。海外のgamified productivity流行（Habitica以降）を、軽量で美しい1人用に再設計。

## 海外事例分析
- Habitica: ゲーム性は強いが重い
- Sunsama: 計画性は高いが可視化演出は弱め
- Finch: 継続設計が上手い

## ターゲット
- ToDoアプリが続かない個人
- 学生/副業ワーカー
- クリエイター

## 料金
- Free: 3クエスト/日
- Solo Pro: **$4/月**（無制限、テーマ追加、週次レポート）

## ユーザーフロー
1. 朝にクエスト3件入力
2. 実行ごとにマップ進行
3. 夜に1行ふりかえり
4. 週次で進捗カード生成

## デザインコンセプト
- JRPG風ミニマップ
- 余白の大きいUI、操作は最小
- 「完了時の気持ちよさ」を優先

## アーキテクチャ
Next.js + Supabase + PWA + framer-motion

## DB設計
- users(id, plan)
- quests(id, user_id, title, difficulty, due_date, done_at)
- daily_logs(id, user_id, reflection, created_at)
- streaks(user_id, current, best)

## コスト見積もり（月）
- Hosting/DB: $5
- AI要約（週次のみ）: $1
- 合計: **$6**

## MVPスコープ
- クエスト登録/完了
- マップ進行表示
- 週次カードPNG

## マーケ計画
- 「今日のマップ」SNSシェア導線
- Notionテンプレ配布→本体へ誘導

## 技術スタック
Next.js, Supabase, PWA, Stripe, Recharts

## リスク
- 飽き: 季節テーマ配布
- 既存ToDoとの差別化不足: マップ演出に集中

## 競合分析
- Habitica: 機能過多
- TickTick: 実務強いが演出弱い
- 差別化: **軽量・美観・1人最適**

## $20達成シナリオ
- Pro($4)×5人 = $20

## ユニットエコノミクス
- ARPU: $4
- 変動費/人: $0.3
- 粗利率: 92.5%
