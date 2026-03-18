# 📦 NicheRadar Repack JP

## 概要
海外Reddit/Product Huntの投稿から、日本市場向けに**「刺さる訴求角度+LP見出し+価格テスト案」**を自動生成するリパッケージSaaS。

## 海外事例分析
- **GummySearch**: Redditの課題抽出が人気。
- **Exploding Topics**: 早期トレンド検知ニーズが強い。
- 日本では英語情報を読めても、訴求への翻訳が難しい層が多い。

## ターゲット
- 個人開発者
- 小規模SaaS運営者
- Web制作会社の新規提案担当

## 料金
- Solo: $9/月（週20トピック）
- Pro: $19/月（週80トピック）

## ユーザーフロー
1. キーワード入力（例: newsletter analytics）
2. 海外投稿を収集・要約
3. 日本語の訴求3案を出力
4. LP見出しと価格仮説をエクスポート

## デザインコンセプト
「市場調査のミキサー」。カードを混ぜるUI、比較しやすい2カラム。

## アーキテクチャ
- Next.js
- Apify + RSS収集
- OpenAI APIで要約/訴求化
- Supabaseで履歴保存

## DB設計
- users(id, plan)
- queries(id, user_id, keyword, created_at)
- sources(id, query_id, source_name, url, score)
- repacks(id, query_id, angle, headline, price_hypothesis)
- exports(id, user_id, format, created_at)

## コスト見積もり（月）
- Apify: $5
- OpenAI API: $6
- Supabase/Hosting: $0〜$10
- 合計: 約$11

## MVPスコープ
- キーワード検索
- 英文投稿の要約
- 訴求3案生成
- CSV出力

## マーケ計画
- 「海外トレンド→日本語訴求30秒」デモ動画
- noteで毎週トレンド無料公開→有料誘導

## 技術スタック
Next.js, Supabase, Apify, OpenAI API

## リスク
- データソース規約変更 → RSS/公開API主体に設計
- 生成の一般論化 → ペルソナ入力欄追加

## 競合分析
- GummySearch: 英語圏特化
- 国内競合: 断片的な翻訳ツールのみ
- 差別化: **日本語訴求まで一気通貫**

## $20達成シナリオ
- Solo $9 × 3人 = $27

## ユニットエコノミクス
- ARPU: $9
- 変動費/ユーザー: $1.8
- 粗利: $7.2 (80%)
