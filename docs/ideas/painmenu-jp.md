# 🧪 PainMenu JP

## 概要
海外Reddit/IndieHackersの悩み投稿を収集し、**日本向けに「売れる機能メニュー」へ変換**するマイクロSaaS。週1配信で実装アイデアを即利用可能にする。

## 海外事例分析
- **GummySearch/Exploding Topics**: 情報発見は強い
- **IndieHackersの手動探索文化**: 再現性が低い
- ギャップ: 日本語で「そのまま作る機能仕様」に落ちるサービスが少ない

## ターゲット
- 個人開発者
- 受託からプロダクト転換したいフリーランス

## 料金
- Lite: $7/月（週3テーマ）
- Pro: $12/月（週10テーマ+CSV）

## ユーザーフロー
1. ニッチ領域を登録
2. 海外投稿を収集
3. 痛みをクラスタ化
4. 日本語の機能メニューで配信

## デザインコンセプト
「**メニュー表UI**」。課題→解決機能→価格例を1枚表示。

## アーキテクチャ
- Python収集バッチ（Reddit API）
- Next.jsダッシュボード
- LLM要約・翻訳パイプライン

## DB設計
- users(id, plan)
- sources(id, type, query)
- pain_items(id, source_id, raw_text, score)
- menus(id, user_id, niche, feature_bundle, price_hint)

## コスト見積もり（月）
- Render Cron: $7
- Supabase: $0
- LLM: $6
- 合計: **$13**

## MVPスコープ
- Reddit 20サブレ限定
- 週次メール配信
- 価格ヒント生成

## マーケ計画
- 「今週の海外痛み3選」をXで公開
- Product Hunt風の無料レポート配布

## 技術スタック
Python, Next.js, Supabase, OpenAI API, Resend, Stripe

## リスク
- API制限 → キャッシュと収集頻度調整

## 競合分析
- GummySearch: 英語圏向け情報探索中心
- 一般トレンドツール: 実装粒度が浅い

## $20達成シナリオ
- Lite 3人 ($21) で達成

## ユニットエコノミクス
- ARPU: $7
- 変動費/人: $1.5
- 粗利率: 78%
