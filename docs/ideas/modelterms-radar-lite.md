# 🛡️ ModelTerms Radar Lite

## 概要
OpenAI/Anthropic/Google等のモデル利用規約・料金・商用利用条項の変更を追跡し、**「プロダクトに影響する変更だけ」**通知する保険型マイクロSaaS。

## 海外事例分析
- TermsMonitor等の汎用規約監視はある
- AIモデル特化で「商用可否/再配布/データ保持」まで要点化するツールは少ない
- 日本ではAI利用が急増する一方、法務チェック体制は弱く機会あり

## ターゲット
- AI機能を提供する小規模SaaS
- 受託開発会社
- 個人開発者

## 料金
- Lite: $5/月（3プロバイダ）
- Pro: $12/月（10プロバイダ + 週次レポート）

## ユーザーフロー
1. 監視対象プロバイダを選択
2. 重要項目タグを設定（価格/商用利用/データ保持）
3. 差分検出
4. AIがリスク要約（高/中/低）
5. Slack/Discord通知

## デザインコンセプト
- 法務ぽさを減らし「運用ダッシュボード」風
- 差分ハイライトを可読重視で表示

## アーキテクチャ
- 取得: 日次スクレイプ + キャッシュ
- 差分: text diff + clause classifier
- 通知: Discord/Slack webhook
- 保存: Supabase

## DB設計
- users(id, plan)
- providers(id, name, tos_url, pricing_url)
- snapshots(id, provider_id, fetched_at, content_hash)
- changes(id, provider_id, severity, summary, section)
- alerts(id, user_id, change_id, channel)

## コスト見積もり（月）
- Supabase: $0〜$25
- クロール/ジョブ: $2
- AI要約: $1.5
- 合計: 約$3.5〜$28.5

## MVPスコープ
- 5社監視
- 差分表示
- Discord通知
- 重要タグフィルタ

## マーケ計画
- AI SaaS開発者コミュニティで配布
- 「規約変更見逃しチェックリスト」配布

## 技術スタック
Node.js, Supabase, Playwright, OpenAI mini, Discord webhook, Stripe

## リスク
- 規約ページ構造変更
- 法的助言と誤認される表現

## 競合分析
- 汎用TOS監視: AI実務に必要な観点が不足
- 差別化: **AIモデル特化 + 実装影響に翻訳**

## $20達成シナリオ
- Lite 4人で$20
- 継続率高め（解約すると監視穴が生まれる）

## ユニットエコノミクス
- ARPU: $5
- COGS/user: $0.4
- 粗利: $4.6（92%）
