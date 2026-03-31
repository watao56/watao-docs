# 🍴 TrendFork JP

## 概要
海外プロダクトのランディングページ/投稿を貼ると、**日本向けの実装仮説（機能・価格・訴求）を1ページ化**するマイクロSaaS。調査→設計の初速を上げる。

## 海外事例分析
- **Exploding Topics / Trends.vc**: 海外トレンド発見の需要
- **GummySearch**: Reddit起点の課題抽出
- **Swipewell/Foreplay**: 訴求ストック文化
- 日本では「発見後の実装化テンプレ」が不足

## ターゲット
- Indie Hacker
- 新規事業担当
- 受託制作の提案担当

## 料金
- Free: 月5解析
- Pro: $7/月（80解析）
- Credit Pack: $3/30解析

## ユーザーフロー
1. URL投入
2. 市場カテゴリ選択
3. 競合/訴求/価格案を自動生成
4. Notion/Markdownへエクスポート

## デザインコンセプト
料理レシピ風UI（材料=要件、手順=実装）。

## アーキテクチャ
- Next.js
- Firecrawl + LLM要約
- Supabase
- Queue（Upstash）

## DB設計
- users(id, plan, credits)
- sources(id, user_id, url, raw_text)
- analyses(id, source_id, category, output_json)
- exports(id, analysis_id, format, url)

## コスト見積もり（月）
- Hosting: $5
- LLM API: $4
- Crawl: $3
- 合計: **$8〜12**

## MVPスコープ
- URL1件解析
- 4項目出力（機能・価格・訴求・MVP）
- Markdown出力

## マーケ計画
- 「海外サービス→日本化」日次投稿
- 起業家コミュニティで無料枠提供
- 解析結果テンプレを配布

## 技術スタック
Next.js / Supabase / OpenAI API / Firecrawl / Upstash

## リスク
- ソース精度依存
- ハルシネーション

## 競合分析
- Trend媒体: 情報提供止まり
- 本案: そのまま実装に落ちる設計書を返す

## $20達成シナリオ
- Pro 3人（$21）で達成

## ユニットエコノミクス
- ARPU: $7
- 変動費/ユーザー: $1.5
- 粗利/ユーザー: $5.5
- 粗利率: 78.5%
