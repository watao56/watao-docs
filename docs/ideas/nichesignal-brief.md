# 🌍 NicheSignal Brief

## 概要
海外コミュニティ（Reddit/Product Hunt/Hacker News）から、**日本で未普及の小さな勝ち筋**を毎日1枚ブリーフで配信するインサイトSaaS。

## 海外事例分析
- **Exploding Topics**: トレンド検知の需要は継続
- **Trends.vc**: 深掘りレポート課金が成立
- **GummySearch**: Reddit起点の課題抽出が伸長
- 日本ギャップ: 英語圏の一次情報を咀嚼する時間がない

## ターゲット
- 個人開発者
- マーケ担当者
- 新規事業の少人数チーム

## 料金
- Free: 週2本
- Pro: **$8/mo**（毎日配信＋検索）
- Team: $24/mo（3席）

## ユーザーフロー
1. トピック（AI、Creator、B2B等）選択
2. 日次で1枚ブリーフ受信
3. 「日本向け実装案」3つを保存
4. Slack/Discordに共有

## デザインコンセプト
"**one-page signal**"。新聞見出し風タイポ＋カード1枚完結。

## アーキテクチャ
- Crawler: scheduled workers
- NLP: embedding + clustering
- App: Next.js
- DB: Postgres
- Delivery: Email/Discord webhook

## DB設計
- users(id, email, plan, interests)
- sources(id, channel, url, fetched_at)
- signals(id, topic, score, summary_en, summary_ja)
- briefs(id, user_id, signal_id, delivered_at)
- saves(id, user_id, brief_id, note)

## コスト見積もり（月）
- Worker/DB: $10
- LLM翻訳要約: $9
- 配信: $1
- 合計: **$20**

## MVPスコープ
- 3ソース連携（Reddit/PH/HN）
- 日次3件抽出
- 日本語1枚ブリーフ

## マーケ計画
- 「今日の海外ニッチ」無料公開枠をX投稿
- Discordコミュニティへのbot配信
- 初月$3オファー

## 技術スタック
Python crawler / Cloudflare Workers / Next.js / Postgres / Stripe

## リスク
- API制限
- 情報鮮度の維持
- 要約品質の再現性

## 競合分析
Exploding Topicsより低価格・高頻度・日本語実装案付き。

## $20達成シナリオ
- Pro 3ユーザー = $24
- コストを無料枠活用で$10以下に圧縮すれば粗利$20近辺

## ユニットエコノミクス
- ARPU: $8
- 変動費/有料ユーザー: $1.5
- 粗利率: 81%
