# 🧭 PulseGarden Daily

## 概要
海外で流行中の「mood-based productivity（Rise/Structured周辺）」を日本語UX化。朝に気分と体力を入力すると、1日のタスクを3色の“庭タイル”で再配置する個人向けツール。

## 海外事例分析
- **Rise**: コンディション連動
- **Structured**: 時間可視化
- **Finch**: 感情に寄り添う継続体験
- 日本市場は“感情可視化×仕事実行”の間に空白がある

## ターゲット
- 在宅ワーカー
- ADHD傾向のある知的労働者
- 1人事業主

## 料金
- Free: 1日5タスク
- Pro: $6/月（無制限+履歴分析）

## ユーザーフロー
1. 朝チェックイン（気分/体力）
2. タスクを自動色分け（集中/軽作業/休憩）
3. 実行後ワンタップ記録
4. 週次で自分の最適リズムを提案

## デザインコンセプト
- ミニマルな庭UI
- 緑系トーン+余白多め
- 触って気持ちいいアニメーション

## アーキテクチャ
- Next.js PWA
- Supabase DB
- 推薦ロジック: ルールベース + 軽量LLM補助

## DB設計
- users(id, email, plan)
- daily_checkins(id, user_id, mood, energy, created_at)
- tasks(id, user_id, title, effort, category, status)
- task_logs(id, task_id, date, completed)
- weekly_insights(id, user_id, week_key, suggestion)

## コスト見積もり（月）
- DB/Hosting: $3
- AI: $1.5
- 合計: $4.5

## MVPスコープ
- チェックイン
- タスク色分け
- 週次サマリ

## マーケ計画
- 「今日の庭スクショ」共有キャンペーン
- 仕事術コミュニティに無料配布

## 技術スタック
- Next.js / Supabase / Recharts
- OpenAI mini系

## リスク
- 習慣化までの離脱
- 推薦精度不足

## 競合分析
- 既存はカレンダー寄り
- PulseGardenは**感情を主軸に実行を再編成**

## $20達成シナリオ
- Pro 4人 = $24/月

## ユニットエコノミクス
- ARPU: $6
- 変動費: $0.6
- 粗利率: 90%
