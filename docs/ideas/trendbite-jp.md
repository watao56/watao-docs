# 📈 TrendBite JP

## 概要
Product Hunt/Reddit/HNの海外トレンドを毎朝5件、**日本向け実装メモ**に変換するマイクロSaaS。情報収集ではなく「明日作れる粒度」に落とす。

## 海外事例分析
- 海外ではトレンドキュレーション系（Trends系ニュースレター）が定着
- 日本は翻訳止まりが多く、実装指示レベルまで降りてこない

## ターゲット
- 個人開発者
- マイクロSaaS運営者
- 企画職

## 料金
- Free: 週2回
- Pro: $7/月（毎日+アラート）
- Goal: 3人で$21

## ユーザーフロー
1. 興味カテゴリ選択
2. 毎朝「海外事例→日本化案」を受信
3. ワンクリックでNotionに保存

## デザインコンセプト
- ニュースではなく「設計カード」UI
- 1カード=課題/仮説/価格/MVP

## アーキテクチャ
- クローラ: scheduled functions
- 要約/変換: OpenAI mini
- 配信: Email + Discord webhook

## DB設計
- users(id, plan, topics)
- sources(id, url, source_type, fetched_at)
- bites(id, topic, jp_summary, mvp_hint)
- deliveries(id, user_id, bite_id, channel)

## コスト見積もり（月）
- Fetch/保存: $2
- AI: $4
- メール: $1
- 合計: $7

## MVPスコープ
- 3ソース連携(PH/Reddit/HN)
- 日次配信
- 保存連携(Notion)

## マーケ計画
- 7日無料トライアル
- 「今日の海外アイデア」X投稿で流入獲得

## 技術スタック
Node.js / Supabase / OpenAI mini / Resend / Stripe

## リスク
- ソース規約変更: RSS/API優先で実装

## 競合分析
- 翻訳ニュースは多い
- 実装粒度まで落とす日本語特化は差別化余地大

## $20達成シナリオ
- トライアル20人→有料15%で3人

## ユニットエコノミクス
- ARPU: $7
- 変動費/人: $1.8
- 粗利率: 74%
