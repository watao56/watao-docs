# 🧩 FormStory Lite

## 概要
Google Form/Tallyの回答を取り込み、AIが**実績・レビュー・活用事例ページ**を自動生成。1人開発の「信頼づくり」を最短化するマイクロSaaS。

## 海外事例分析
- Senja: testimonial収集需要が急成長
- Typedream/Framer: LP制作は強いが素材整理が手間
- Tally: 回答収集はできるが公開活用が弱い

## ターゲット
- 個人開発者
- 小規模受託
- コンテンツ販売者

## 料金
- Free: 1公開ページ
- Maker: $7/月（3ページ、カスタムドメイン）
- Studio: $15/月（10ページ、SEO設定）

## ユーザーフロー
1. Form連携
2. 回答をAIでタグ/要約
3. テンプレ選択
4. 公開URL発行

## デザインコンセプト
「**証拠が主役のミニLP**」。余計な装飾を削り、信頼情報をカード化。

## アーキテクチャ
- Next.js + ISR
- Supabase Postgres
- Integrations: Google Forms API / Tally Webhook
- AI分類: OpenAI mini
- Hosting: Vercel

## DB設計
- users(id, plan)
- sources(id, user_id, provider, token_ref)
- responses(id, source_id, author, body, rating, tags_json)
- pages(id, user_id, slug, theme, seo_json, published)

## コスト見積もり（月）
- Vercel/Supabase無料枠〜$0
- AI分類 $1〜$4
- 合計: **$1〜$6**

## MVPスコープ
- Google Form連携
- 自動タグ付け
- 3テンプレ表示
- カスタムドメイン（Pro）

## マーケ計画
- 「公開レビュー100件テンプレ」無料配布
- Product Hunt日本勢向け導入記事
- テンプレギャラリーで自然流入

## 技術スタック
Next.js / Supabase / Vercel / OpenAI mini / Stripe

## リスク
- API仕様変更 → CSVインポートを代替手段に
- SEO弱さ → 構造化データを標準搭載

## 競合分析
Testimonial収集ツールは多いが、FormStory Liteは**既存フォーム資産をそのまま公開価値化**できる。

## $20達成シナリオ
Maker($7)×3人で$21達成。

## ユニットエコノミクス
- ARPU: $7
- 変動費: $0.6/人
- 粗利: 約91%
