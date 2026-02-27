# 🛡️ Creator Rule Radar

## 概要
🛡️ Creator Rule Radarは、保険型（1/5のみ）: プラットフォーム規約変更アラートに特化した月$20達成向けマイクロプロダクト。AIエージェントのみで2週間MVP実装を前提に、低コスト運用で黒字化を狙う。

## 海外事例分析
- 参照トレンド: YouTube/TikTok収益化条件変更で収益毀損する事例、ToS監視系需要
- 示唆: 海外では「汎用ツール」か「高価格SaaS」が主流。日本向けには**狭い用途+日本語UX最適化**が刺さる。
- 日本でのギャップ: 使い方が難しい/英語前提/日本文化の導線不足。

## ターゲット
YouTube/TikTok運用者、MCN、小規模クリエイター事務所

## 料金
Solo $8 / Studio $19

## ユーザーフロー
①監視対象プラットフォーム選択→②規約差分抽出→③重要度判定→④「何を直すべきか」チェックリスト通知

## デザインコンセプト
レーダーUI＋差分ハイライト。危険度を色で直感表示。

## アーキテクチャ
Scheduler(cron) + fetch diff + GPT mini classification + Discord/Email notify

## DB設計
主要テーブル: users, watched_policies, snapshots, diffs, alerts, acknowledgements

## コスト見積もり
有料5人時 月$4.2（LLM分類$2.8含む）

## MVPスコープ
日次クロール、差分表示、重要通知、修正TODO生成

## マーケ計画
クリエイター向けコミュニティで「今週の規約変更まとめ」無料配信

## 技術スタック
Node.js, Playwright(optional), Postgres, OpenAI mini, Resend

## リスク
誤検知/見逃し。重大通知には「原文リンク+免責」を必須化。

## 競合分析
汎用Terms監視より「クリエイター収益規約」に絞り、行動提案まで出す。

## $20達成シナリオ
Solo $8×3人 = $24/月

## ユニットエコノミクス
ARPU $8、変動費/人 $0.82、粗利率89.7%
