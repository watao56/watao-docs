# 📦 TinyCase Localizer

## 概要
海外SaaSの成功LP・レビュー・事例を入力すると、日本市場向けに**「実装可能な小さな勝ち筋カード」**を返すマイクロSaaS。単なる翻訳ではなく、価格・導線・表現を日本仕様に圧縮する。

## 海外事例分析
- **GummySearch**: 海外コミュニティの痛み抽出
- **Exploding Topics**: 伸び始め領域の早期発見
- **Taplio/Typefully**: ネタ→配信導線の高速化

## ターゲット
- Indie Hacker
- 受託開発の新規事業担当
- マイクロSaaS個人開発者

## 料金
- Lite: $5/月（30カード）
- Pro: $12/月（200カード）

## ユーザーフロー
1. URL/レビューを投入
2. カード形式で「誰に・何を・どう売るか」生成
3. そのままNotion/Markdownへ出力

## デザインコンセプト
- 研究ノート風UI
- 白黒ベース+差し色1色
- カードを横断比較しやすいレイアウト

## アーキテクチャ
- Next.js
- Web fetch + LLM要約
- Supabase

## DB設計
- users(id, email, plan)
- sources(id, user_id, url, source_type, fetched_text)
- insight_cards(id, source_id, segment, hook, pricing_hint, channel_hint)
- exports(id, user_id, format, created_at)

## コスト見積もり（月）
- Hosting/DB: $3
- AI: $4
- 合計: $7

## MVPスコープ
- URL解析
- 3種類のカード出力
- Markdown export

## マーケ計画
- 「海外SaaS1本を日本向けに再設計」連載
- 週1で無料公開カードを配布

## 技術スタック
- Next.js / Supabase / OpenAI mini

## リスク
- ソース品質依存
- 著作権配慮（引用範囲）

## 競合分析
- 翻訳系ツールは多いが、**日本向け実装カード化**に特化した競合は薄い

## $20達成シナリオ
- Lite 4人 = $20

## ユニットエコノミクス
- Lite ARPU: $5
- 変動費: $0.7
- 粗利率: 86%
