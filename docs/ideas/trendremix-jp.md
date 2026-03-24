# 🌍 TrendRemix JP

## 概要
海外で伸びているショート動画/広告表現を収集し、日本語の文脈に合わせた**使えるリミックス案**を毎日3本提案。AI×クリエイティブのローカライズ特化。

## 海外事例分析
- Foreplay/AdSpy系: 収集は強いが日本向け変換が弱い。
- 海外トレンドの直輸入は文化差で失敗しやすい。
- 「翻訳」ではなく「文脈変換」に価値がある。

## ターゲット
- 1人マーケ担当
- D2C運営者
- 動画編集フリーランス

## 料金
- Starter: **$5/月**（日3提案）
- Solo: **$9/月**（日10提案 + CSV）

## ユーザーフロー
1. 業界を選ぶ
2. 海外事例カードを見る
3. 日本向けリミックス案を取得
4. そのまま台本テンプレに変換

## デザインコンセプト
- ニュースルーム×カードUI
- 1画面完結で意思決定

## アーキテクチャ
- ETL: GitHub Actions + RSS/API
- 分析: OpenAI Responses API
- App: Next.js + Supabase

## DB設計
- sources(id, name, type)
- trend_items(id, source_id, url, summary)
- remixes(id, trend_item_id, jp_angle, script)
- users(id, plan)

## コスト見積もり（月）
- Hosting/DB: $0〜$5
- AI推論: $8
- 合計: **$13**

## MVPスコープ
- 5ソース連携
- 日次提案生成
- 台本テンプレ出力
- 課金

## マーケ計画
- 「海外→日本リミックス」日刊ポスト
- 代理店コミュニティで導入検証

## 技術スタック
Next.js / Supabase / GitHub Actions / OpenAI API / Stripe

## リスク
- ソース規約変更
- 提案品質のブレ

## 競合分析
- AdSpy系: 監視特化
- 本案: **実行可能な日本向け変換まで提供**

## $20達成シナリオ
- Starter $5 × 4人 = $20

## ユニットエコノミクス
- ARPPU: $5
- 変動費: $1.5
- 粗利率: 70%
