# 🔗 LinkMenu Lite

## 概要
クリエイター向けに、**プロフィールリンクページを“イベント/販売導線”中心で自動最適化**するマイクロSaaS。海外のLink-in-bio市場は成熟しているが、日本語UIと販売導線分析が弱い。

## 海外事例分析
- Linktree/Beacons: 巨大市場、機能多すぎ問題
- Bento: デザイン重視
- 日本ギャップ: 少機能で速い運用＋和文最適コピー提案

## ターゲット
- 小規模クリエイター
- ハンドメイド販売者
- セミナー主催者

## 料金
- Free: 1ページ、基本テーマ
- Pro: $5/月（ABテスト、AIコピー提案）
- Shop: $9/月（商品カードテンプレ）

## ユーザーフロー
1. SNSと販売先URLを登録
2. AIがCTA順序を提案
3. 1クリックで公開
4. 週次でクリック分析を確認

## デザインコンセプト
- 「カードスタック」
- モバイル優先、余白広め、ブランド色強調

## アーキテクチャ
- Next.js static rendering
- 計測: Cloudflare Workers
- AIコピー提案: gpt-4o-mini

## DB設計
- users(id, email, plan)
- pages(id, user_id, slug, theme, published)
- links(id, page_id, label, url, order_no)
- clicks(id, page_id, link_id, ts, referrer)
- experiments(id, page_id, variant_a, variant_b, winner)

## コスト見積もり（月）
- Hosting: $0
- Workers/KV: $1
- AI: $1
- 合計: **約$2**

## MVPスコープ
- ページ作成
- クリック計測
- CTA文面提案

## マーケ計画
- 「1ページ改善でクリック率+xx%」事例投稿
- テンプレ無料配布（美容/講師/ハンドメイド）
- マイクロインフルエンサーに無償提供

## 技術スタック
Next.js / Cloudflare Workers / KV / Supabase / Stripe

## リスク
- Linktreeとの比較で埋没
- 計測のプライバシー配慮

## 競合分析
- Linktreeより軽量・安価
- BentoよりAIコピー提案で運用工数削減

## $20達成シナリオ
- Pro 4人で $20 MRR

## ユニットエコノミクス
- ARPU: $5
- 変動費/人: $0.1
- 粗利率: 98%
