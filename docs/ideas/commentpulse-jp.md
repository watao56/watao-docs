# 💬 CommentPulse JP

## 概要
TikTok/Instagram/YouTubeの投稿URLを入れると、コメントを収集し**刺さった訴求・不満・購買トリガー**を日本語でカード化するマイクロSaaS。海外のソーシャルリスニングの軽量版。

## 海外事例分析
- コメント分析系はSprout Social等が高価格帯で提供
- D2Cでは「コメントから広告フック抽出」が定番化
- 日本ではBI寄りで高額、**個人〜小規模向けの低価格特化が不足**

## ターゲット
- SNS運用代行
- D2Cブランド担当
- 個人クリエイター

## 料金
- Lite: $6/月（30投稿）
- Pro: $14/月（200投稿）

## ユーザーフロー
1. 投稿URLを登録
2. コメント取得（API/手動CSV）
3. AIが感情/意図/購買障壁を分類
4. 「次に使うべき訴求3つ」を提示
5. Notion/CSVへ出力

## デザインコンセプト
- 「会話の温度が見える分析UI」
- ヒートマップ + 要約カード

## アーキテクチャ
- Next.js
- Workerで収集ジョブ
- NLP分類は低コストLLM + ルール
- ストレージはSupabase

## DB設計
- users(id, plan)
- sources(id, user_id, platform, url)
- comments(id, source_id, author_hash, text, created_at)
- insights(id, source_id, angle, evidence_count)
- exports(id, user_id, format, url)

## コスト見積もり（月）
- Supabase: $0〜$25
- AI分類: $4
- Worker: $2
- 合計: 約$6〜$31

## MVPスコープ
- URL登録
- コメント分類
- 訴求カード表示
- CSV出力

## マーケ計画
- 運用代行向け「週次レポート自動化」訴求
- 先着20ユーザーにテンプレ配布

## 技術スタック
Next.js, Supabase, Cloudflare Workers, OpenAI mini, Stripe

## リスク
- API仕様変更
- コメント取得制限

## 競合分析
- Sprout Social等: 高機能だが高価格
- 差別化: **低価格・訴求抽出特化・日本語最適化**

## $20達成シナリオ
- Lite 4人で$24
- 粗利80%前後維持

## ユニットエコノミクス
- ARPU: $6
- COGS/user: $1.1
- 粗利: $4.9（81.6%）
