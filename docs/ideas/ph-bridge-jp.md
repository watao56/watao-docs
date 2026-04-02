# 🌉 PH Bridge JP

## 概要
Product Hunt新着プロダクトを収集し、**日本市場向け実装チケット（何を作るか/どう売るか）**に変換するマイクロSaaS。単なる要約でなく、開発タスクまで落とす。

## 海外事例分析
- Product Hunt, BetaList, Uneedのデイリー流量
- Exploding Topics系は抽象度が高く、実装への距離がある
- 日本側は「翻訳はあるが意思決定材料が不足」

## ターゲット
- 週末開発者
- 受託→自社SaaS移行層
- 小規模開発会社の新規事業担当

## 料金
- Free: 1日3件
- Pro: $9/月（無制限、CSV/Notion出力）
- Agency: $19/月（チーム共有）

## ユーザーフロー
1. カテゴリ選択（AI/DevTools/Creator等）
2. 新着案件をカード閲覧
3. 「JP適用度」「開発難易度」「初月販売導線」を確認
4. 1クリックでMVP Backlog生成

## デザインコンセプト
- 「Signal Terminal」
- 白黒ベース+アクセント1色、投資ダッシュボード風

## アーキテクチャ
- Front: Next.js
- Ingest: GitHub Actions cron + official RSS/API
- NLP: LLMでチケット化
- Storage: Supabase

## DB設計
- sources(id, name, url, fetched_at)
- products(id, source_id, title, url, tags, raw_text)
- jp_cards(id, product_id, fit_score, build_scope, pricing_hint)
- users(id, email, plan)

## コスト見積もり（月）
- Hosting/DB: $10
- LLM要約: $6
- 合計: **$16**

## MVPスコープ
- 主要3ソース取り込み
- 日本向け実装カード
- 保存/エクスポート

## マーケ計画
- 「今日の海外SaaSを日本で作るなら」連載投稿
- 開発コミュニティで無料枠提供

## 技術スタック
Next.js, Supabase, GitHub Actions, OpenAI API, Stripe

## リスク
- 情報源依存 → 複数ソース化
- 要約品質 → 手動評価フィードバックループ

## 競合分析
- ニュースレターは“読むだけ”
- 本案は**実装チケット出力**までが差別化

## $20達成シナリオ
- Pro $9 × 3 = $27

## ユニットエコノミクス
- ARPPU: $9.4
- 変動費/有料ユーザー: $1.8
- 粗利率: 約81%
