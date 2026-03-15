# 🌍 NicheHook Pulse JP

## 概要
海外のReddit/Product Hunt/X投稿から、ニッチ市場で刺さっている訴求を抽出し、**日本語LP向けフック文・見出し案**を毎日5本生成するマイクロSaaS。既存の翻訳ツールでは得られない“売れる言い回し”の局所最適に特化。

## 海外事例分析
- GummySearch: Reddit洞察抽出が人気
- Exploding Topics: トレンド把握は強いが実装に落ちない
- SwipeWell/Foreplay: 広告収集は強いが和文化は弱い
- 日本ギャップ: 海外トレンド→日本語LP化の翻訳コストが高い

## ターゲット
- LP運用する個人事業主
- 小規模マーケ代理店
- SaaSの一人マーケ担当

## 料金
- Free: 1業界、日次2フック
- Solo: $8/月（3業界、日次15フック）
- Studio: $19/月（10業界、CSV/API）

## ユーザーフロー
1. 業界キーワードを登録（例: 英会話アプリ）
2. 毎日スクレイプ結果を要約
3. AIが日本語フック5〜15本生成
4. LP見出し/CTAとしてコピー
5. 成果メモを残して次回改善

## デザインコンセプト
- **“Signal Dashboard”**
- 黒背景に蛍光ハイライト
- “今日の当たりフック”をカードで強調

## アーキテクチャ
- Collect: Apify or RSS + 手動ソース登録
- Process: Python worker
- LLM: 安価なminiモデル
- App: Next.js
- DB: Supabase

## DB設計
- users(id, email, plan)
- niches(id, user_id, keyword, locale)
- source_posts(id, niche_id, source, title, url, score)
- hooks(id, niche_id, date, angle, hook_text, cta_text)
- performance_notes(id, hook_id, user_note, outcome)
- subscriptions(id, user_id, status)

## コスト見積もり（月）
- Hosting/Supabase: $0
- 収集基盤（無料枠中心）: $2
- LLM: $3
- 合計: **約$5/月**

## MVPスコープ
- 3ソース固定（Reddit/PH/X）
- 日次バッチ
- 日本語フック5本生成
- CSV出力

## マーケ計画
- 「海外投稿→和文LP見出し」比較ポストを毎日公開
- 代理店向けに3業界無料診断
- Product Huntの国内コミュニティで配布

## 技術スタック
Next.js / Python / Supabase / Stripe / Apify / OpenAI mini

## リスク
- ソース依存によるデータ欠損
- コピーの品質が業界でぶれる

## 競合分析
- 翻訳ツール: 言語変換のみ → **訴求変換**で差別化
- トレンドツール: 分析止まり → **LP用コピー即利用**まで提供

## $20達成シナリオ
- Solo 3人（$24 MRR）
- または Studio 2人（$38 MRR）

## ユニットエコノミクス
- ARPU: $10.1
- 変動費/人: $0.8
- 粗利/人: $9.3
- 粗利率: **92.1%**
