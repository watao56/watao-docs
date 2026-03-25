# 🌡️ FocusThermostat OS

## 概要
「気温」ではなく**集中温度（Low/Med/High）**でタスクを並べ替える個人生産性ツール。起動時10秒チェックで今日の最適順を自動提案。

## 海外事例分析
- **Rise/Structured/Sunsama**: 時間ブロック管理が主流
- **Finch**: 感情入力で継続性を向上
- ギャップ: 日本語で“体調×集中温度”を即タスク化する軽量体験が少ない

## ターゲット
- 予定通り動けないリモートワーカー
- 副業で時間が不規則な個人

## 料金
- Free: 1日20タスク
- Plus: $4/月（履歴分析、自動再配置、Notion同期）

## ユーザーフロー
1. 朝に温度入力
2. タスク自動並び替え
3. 完了で温度学習
4. 夜に振り返り1枚カード

## デザインコンセプト
「**温度計UI**」。赤青グラデーションで認知負荷を下げる。

## アーキテクチャ
- PWA (Next.js)
- Supabase
- OpenAI API（夜の振り返り要約のみ）

## DB設計
- users(id, timezone, plan)
- tasks(id, user_id, title, effort, due_at, status)
- temp_logs(id, user_id, level, note, created_at)
- daily_reports(id, user_id, summary, score)

## コスト見積もり（月）
- Hosting: $0
- Supabase: $0
- LLM要約: $2
- 合計: **$2**

## MVPスコープ
- 温度入力3段階
- 自動ソート
- 日次レポート

## マーケ計画
- 「今日はLowでも回るタスク術」コンテンツ発信
- Notionテンプレ配布で流入

## 技術スタック
Next.js PWA, Supabase, OpenAI API, Stripe

## リスク
- 習慣化失敗 → 10秒入力＆通知を最小に

## 競合分析
- Sunsama: 高機能だが価格高め
- Finch: 習慣寄りで仕事タスク最適化は弱い

## $20達成シナリオ
- Plus 5人 × $4 = $20/月

## ユニットエコノミクス
- ARPU: $4
- 変動費/人: $0.2
- 粗利率: 95%
