# 🌱 TaskBloom Loop

## 概要
「今日の気分・体力・空き時間」から、タスクを**3レイヤー（攻め/維持/回復）**に自動並べ替えるパーソナル生産性ツール。

## 海外事例分析
- **Rise / Rize**: 体調・集中状態の可視化
- **Sunsama**: 1日計画の実行率改善
- **Finch**: 感情入力の継続性が高い

## ターゲット
- 副業クリエイター
- ADHD傾向でタスク優先が崩れやすい層

## 料金
- Free: 1日プランまで
- Plus: $7/月

## ユーザーフロー
1. 朝に気分スライダー入力
2. 今日の時間枠を入力
3. AIが3レイヤーで再配置
4. 終了時に「実績花びら」を生成

## デザインコンセプト
「**Living Petal UI**」: 実行すると花びらが増える可視化。SNS共有向け。

## アーキテクチャ
- PWA (Next.js)
- Supabase Auth/DB
- OpenAI mini for ranking
- cronで日次まとめ配信

## DB設計
- users(id, tz, plan)
- moods(id, user_id, energy, focus, logged_at)
- tasks(id, user_id, title, est_min, lane)
- sessions(id, user_id, planned_min, done_min)

## コスト見積もり（月）
- Supabase: $0
- AI: $2（有料20人）
- 合計: **$2〜$4**

## MVPスコープ
- 気分入力
- 3レーン自動配置
- 1日サマリー画像生成

## マーケ計画
- #今日の花びら 投稿テンプレ
- Notionテンプレ配布導線

## 技術スタック
Next.js PWA, Supabase, OpenAI mini, Vercel

## リスク
- 三日坊主 → 7日連続バッジ
- AI提案がずれる → 手動オーバーライド導線

## 競合分析
- Sunsama: 高機能だが重い
- Finch: 生産性の直接導線が弱い

## $20達成シナリオ
- Plus 3人 = $21

## ユニットエコノミクス
- ARPU: $7
- 変動費: $0.2
- 粗利率: **97%**
