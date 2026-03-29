# 🌏 TrendCapsule JP

## 概要
TrendCapsule JPは、英語圏ニュースレター/Reddit/YouTubeから「日本未浸透の小さな勝ち筋」を抽出し、**日本向け実装カード（課金導線付き）**に変換するマイクロSaaS。情報収集ではなく「次アクション」まで落とす。

## 海外事例分析
- **Exploding Topics / Trends.vc**: 発見は強いが実装手順が薄い。
- **GummySearch**: Reddit発掘に強いが日本語運用に壁。
- 余地: 日本語で「誰にどう売るか」まで定型化。

## ターゲット
- 個人開発者
- 受託→自社SaaSへ移行したいエンジニア
- 小規模マーケ事業者

## 料金
- Lite: $5/月（週3カード）
- Builder: $10/月（毎日1カード + CSV）

## ユーザーフロー
1. 興味領域選択（EC/教育/採用など）
2. 毎日1枚のトレンドカード受信
3. 「日本での最初の顧客像」「最小MVP」確認
4. 1クリックでNotion/Trelloへ転送

## デザインコンセプト
- **Signal Terminal**: ダーク基調+シグナル色
- “情報メディア”より“意思決定端末”の見た目

## アーキテクチャ
- Crawler: Apify + RSS
- NLP: OpenAI for clustering/summarization
- App: Next.js + Supabase
- Delivery: Email + Discord webhook

## DB設計
- sources(id, type, url, lang)
- signals(id, source_id, title, summary_en, score)
- jp_cards(id, signal_id, persona_jp, offer_jp, mvp_jp)
- users(id, email, plan, niches)
- deliveries(id, user_id, card_id, channel, delivered_at)

## コスト見積もり（月）
- Apify: $5
- Supabase: $0
- OpenAI: $6〜$12
- Email送信: $1
- **合計: $12〜$18**

## MVPスコープ
- ソース20件固定
- 日次5カード生成
- メール配信
- Notion連携

## マーケ計画
- Indie Hackersで英語版抜粋を先行配布
- 日本語Xで「本日の未上陸トレンド」毎日投稿
- 初月は手動オンボーディング10人

## 技術スタック
Next.js, Supabase, Apify, OpenAI, Resend, Discord webhook

## リスク
- 情報の鮮度劣化 → ソース週次見直し
- 誤検出 → human-in-the-loopで最終承認

## 競合分析
- ニュースレター: 読んで終わる
- TrendCapsule JP: 収益化導線（価格・MVP）まで提示

## $20達成シナリオ
- Builder 2人（$20）で到達
- Lite 4人（$20）でも達成

## ユニットエコノミクス
- ARPU: $6.7
- 変動費: $2.0
- 粗利率: 70%
