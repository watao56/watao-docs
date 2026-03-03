# 🗺️ Reply2Roadmap

## 概要
X/Discord/フォーム返信を収集し、**公開ロードマップ候補カード**に自動変換するマイクロSaaS。作る前の需要検証を高速化し、Build in Publicの投稿素材まで自動生成。

- カテゴリ: マイクロSaaS（Indie Hacker向け）
- 目標: 月$20（$10プラン×2人）

## 海外事例分析
- Canny: 要望管理は強いが初期導入が重い
- Productboard: 中規模以上向けで高価格
- Indie Hackers文化: 軽量な「公開進捗」の需要が高い

示唆: 日本の個人開発者には「重い機能」より**即投稿できる軽さ**が有効。

## ターゲット
- 個人開発者
- 小規模SaaS運営者
- X中心で顧客ヒアリングする人

## 料金
- Free: 30件/月取り込み
- Solo: $10/月（500件、公開ボード、週次レポート）
- Maker+: $19/月（2プロダクト、投票、優先度AI分類）

## ユーザーフロー
1. X/Discordを接続
2. 返信データを自動取り込み
3. AIが要望カード化（重複統合）
4. 「Now/Next/Later」に自動配置
5. 共有URLをそのままポスト

## デザインコンセプト
- カンバンUIを極限まで軽量化
- 1クリックでカードをSNS画像化
- 「作ってる感」が出る進捗バー

## アーキテクチャ
- Next.js + Supabase
- Fetcher: Cron + provider API
- AI分類: embedding + miniモデル
- 画像化: Satori + Resvg

## DB設計
- sources(id, user_id, source_type, token_ref)
- raw_feedback(id, source_id, text, author, posted_at)
- idea_cards(id, user_id, title, summary, status, score)
- merges(id, from_card_id, to_card_id, reason)

## コスト見積もり（月）
- インフラ: $0〜$5
- AI分類: $1〜$4
- 合計: $2〜$9

## MVPスコープ
- X/Discordの2入力源
- カード化 + 重複統合
- 公開ボードURL
- 画像書き出し

## マーケ計画
- 「返信100件→次作の機能3つ」実録投稿
- #buildinpublicタグ運用
- 無料テンプレ（ロードマップ画像）配布

## 技術スタック
Next.js / Supabase / OpenAI mini / Cron / Satori / Stripe

## リスク
- API仕様変更
- 誤分類による信頼低下
- 公開ボード炎上リスク

## 競合分析
- Canny/Productboardは導入重量級
- 本案は**個人開発向けに超軽量化**し即日導入を狙う

## $20達成シナリオ
- Solo $10 × 2人で即達成
- 無料50ユーザー→有料4%で到達

## ユニットエコノミクス
- ARPU: $11.8
- 変動費: $1.3/user
- 粗利: $10.5/user（89%）
- LTV改善: 公開ボードが継続運用資産になる
