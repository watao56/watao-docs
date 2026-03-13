# 🌍 TrendLocalizer Lite

## 概要
海外でバズるLP/広告クリエイティブを収集し、日本市場向けの**訴求角度・見出し・CTA**にローカライズして返すマイクロSaaS。

## 海外事例分析
- Foreplay / SwipeWell: 広告スワイプ保存が定着
- Exploding Topics: 海外シグナル収集の需要
- Typefully: “そのまま使える”テンプレ課金が強い

## ターゲット
- D2C運営者、広告代理店、個人マーケター

## 料金
- Starter: $9/月（週20分析）
- Pro: $19/月（週100分析）

## ユーザーフロー
1. URL貼付 or キーワード入力
2. 海外事例3件抽出
3. 日本語コピー案/訴求マップを生成
4. LP案としてエクスポート

## デザインコンセプト
「Radar + Tearsheet」。分析感と実用性を両立。

## アーキテクチャ
Workerでスクレイプ、LLMで要約、Supabase保存。法的配慮で本文は最小キャッシュ。

## DB設計
- users(id, plan)
- scans(id, user_id, keyword, market)
- examples(id, scan_id, source_url, angle)
- outputs(id, scan_id, jp_headline, cta, proof)

## コスト見積もり（月）
- Hosting: $0
- AI: $6
- Proxy/Fetch: $2
- 合計: 約$8

## MVPスコープ
- URL3件比較
- 日本語訴求生成
- CSV出力

## マーケ計画
- 「海外LP→日本語3案」ビフォーアフター投稿
- 広告運用コミュニティへの体験配布

## 技術スタック
Cloudflare Workers, Supabase, Next.js, OpenAI mini

## リスク
- 著作権/利用規約 → 引用最小化、URL参照中心

## 競合分析
既存は保存止まり。TrendLocalizerは“日本語で即使える”までを担保。

## $20達成シナリオ
- Starter 3人（$27）で達成

## ユニットエコノミクス
- ARPU: $11
- 変動費/人: $1.2
- 粗利率: 89%
