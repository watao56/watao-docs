# 🍱 SignalSnack JP

## 概要
海外Reddit/HN/Product Huntの伸びスレを収集し、**日本向け実行カード（誰に何を売るか）**に圧縮する1人起業家向けマイクロSaaS。

## 海外事例分析
- GummySearch: Reddit探索は強いが日本語化不足
- Exploding Topics: マクロ傾向は掴めるが実装粒度が粗い
- 示唆: 日本では「今週何を作るか」まで落ちる具体性が必要

## ターゲット
- Indie Hacker
- 週末開発者
- 受託から自社プロダクトに移りたい層

## 料金
- Free: 週3カード
- Solo: $7/月（週20カード）
- Builder: $15/月（CSV/API）

## ユーザーフロー
1. 興味タグ選択
2. 毎朝シグナルカード配信
3. 気になるカードを保存
4. 1クリックで「検証タスク」に変換

## デザインコンセプト
「Bento Intelligence」：カード式、余計な分析を削った即断UI。

## アーキテクチャ
- Cron収集（Reddit/HN APIs）
- NLP要約 + 日本語ローカライズ
- Next.js配信ダッシュボード

## DB設計
- sources(id, platform, post_id, score)
- insights(id, source_id, summary_ja, opportunity_score)
- users(id, tags, plan)
- saves(id, user_id, insight_id, status)

## コスト見積もり（月）
- API/収集: $3
- LLM要約: $6
- Hosting: $5
- 合計: 約$14

## MVPスコープ
- Reddit/HNのみ
- 1日10カード
- 保存＋メモ

## マーケ計画
- 「海外で伸びるSaaS速報」X運用
- 週次ニュースレター連動
- Product Hunt実況の日本語まとめ配信

## 技術スタック
Python collector / Supabase / Next.js / OpenAI mini

## リスク
- API仕様変更 → 収集元を複線化
- ノイズ増加 → スコア閾値と手動ブラックリスト

## 競合分析
情報提供で終わらず、実装タスクに落ちる点で差別化。

## $20達成シナリオ
Solo($7)×3人 = $21/月

## ユニットエコノミクス
- ARPU: $7.3
- 変動費/人: $1.5
- 粗利: $5.8（79.4%）
