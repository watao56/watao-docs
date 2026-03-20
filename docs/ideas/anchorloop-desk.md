# 🧭 AnchorLoop Desk

## 概要
「集中が切れてから戻る」までを設計する、復帰特化のパーソナルOS。離席・通知・SNS脱線を検知し、**30秒の復帰ブリーフ**を自動表示して再着火を早める。

## 海外事例分析
- Sunsama/Routine: 計画は強いが復帰導線は弱い
- Rise: カレンダー最適化中心
- 示唆: 日本ユーザーには“完璧計画”より“戻れる仕組み”が効く

## ターゲット
- 在宅ワーカー
- ADHD傾向で切替に苦労する人

## 料金
- Free: 1日3回まで復帰ブリーフ
- Plus: $4/月（無制限＋週次分析）
- Coach: $9/月（行動レポート共有）

## ユーザーフロー
1. 今日の主要タスク3件を登録
2. 離脱検知（手動/ブラウザ拡張）
3. 復帰時に30秒ブリーフ表示
4. 夜に復帰成功率サマリー

## デザインコンセプト
「Calm Ops」：静かな配色、余白多め、1画面1意思決定。

## アーキテクチャ
- PWA + Browser Extension
- Supabase + Edge Functions
- 軽量LLMで復帰文面生成

## DB設計
- users(id, timezone, plan)
- tasks(id, user_id, title, priority)
- interruptions(id, user_id, started_at, reason)
- recovery_logs(id, interruption_id, success, latency_sec)

## コスト見積もり（月）
- Supabase: $0〜10
- LLM API: $2
- Hosting: $0〜5
- 合計: 約$7

## MVPスコープ
- 手動離脱記録
- 復帰ブリーフ生成
- 日次レポート

## マーケ計画
- Xで「復帰30秒チャレンジ」を公開
- Notionテンプレ配布で流入
- ADHDコミュニティへケース投稿

## 技術スタック
Next.js / Chrome Extension / Supabase / OpenAI mini

## リスク
- 検知精度不足 → MVPは手動優先
- 継続率低下 → 7日連続復帰バッジ

## 競合分析
ToDo管理でなく“再集中速度”に特化したポジショニング。

## $20達成シナリオ
Plus($4)×5人 = $20/月

## ユニットエコノミクス
- ARPU: $4.8
- 変動費/人: $0.5
- 粗利: $4.3（89.5%）
