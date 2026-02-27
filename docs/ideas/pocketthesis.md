# 🧠 PocketThesis

## 概要
🧠 PocketThesisは、パーソナルツール/生産性（読んだ情報の要点化）に特化した月$20達成向けマイクロプロダクト。AIエージェントのみで2週間MVP実装を前提に、低コスト運用で黒字化を狙う。

## 海外事例分析
- 参照トレンド: Readwiseのハイライト再活用、Mymind系の軽量知識整理トレンド
- 示唆: 海外では「汎用ツール」か「高価格SaaS」が主流。日本向けには**狭い用途+日本語UX最適化**が刺さる。
- 日本でのギャップ: 使い方が難しい/英語前提/日本文化の導線不足。

## ターゲット
記事を保存するが見返さない知的労働者、学生、PM

## 料金
Free / Pro $7（無制限要約+自動タグ）

## ユーザーフロー
①URL保存→②3行要約+反対意見+次アクション提案→③週次で「今週の学び」自動生成

## デザインコンセプト
新聞レイアウト風カード。白黒+アクセント1色で高級感。

## アーキテクチャ
Cloudflare Workers + Supabase + Jina Reader/Web fetch + OpenAI mini

## DB設計
主要テーブル: users, captures, summaries, tags, weekly_digests, subscriptions

## コスト見積もり
有料5人時 月$3.6（LLM $2.4, DB/egress $1.2）

## MVPスコープ
URL保存、3行要約、週次メール、タグ検索

## マーケ計画
「積読ゼロ」訴求でNotionテンプレ配布、Product Hunt mini launch

## 技術スタック
Workers, Hono, Supabase, Resend, OpenAI mini

## リスク
要約品質のばらつき。ドメイン別テンプレで改善。

## 競合分析
Readwiseは強力だが重い。PocketThesisは「3行＋行動」特化の軽さで差別化。

## $20達成シナリオ
Pro $7×3人 = $21/月

## ユニットエコノミクス
ARPU $7、変動費/人 $0.48、粗利率93%
