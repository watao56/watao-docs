# 💳 PricePatch Radar

## 概要
海外SaaSで急増している「価格ページ改定・無料枠縮小」を監視し、**使っているSaaSの値上げ兆候だけ**を通知する軽量ツール。保険型は本案1件のみ。

## 海外事例分析
- **Visualping / Hexowatch**: ページ差分監視需要は実証済み
- ただし汎用監視はノイズが多い。
- 日本では「SaaS料金改定」に特化し、影響額まで出すサービスは希少。

## ターゲット
- 個人開発者、小規模SaaS運営、経理兼務フリーランス

## 料金
- Lite: **$3/月**（10サービス）
- Pro: $7/月（50サービス + 月次損益レポート）

## ユーザーフロー
1. 利用中SaaSを選択
2. 価格ページ差分を毎日検出
3. 値上げ/無料枠改定のときだけ通知
4. 月次で「追加コスト見込み」提示

## デザインコンセプト
- 家計簿×レーダーUI
- 変化があった時だけ赤アクセントで目立たせる

## アーキテクチャ
- Scheduled worker (GitHub Actions + Cloudflare Worker)
- Diff engine (Playwright + text normalization)
- DB: Supabase
- 通知: Discord / Email

## DB設計
- users, tracked_services, snapshots, diffs, alerts, monthly_impact

## コスト見積もり（月）
- Cloudflare free: $0
- Supabase: $0
- AI分類（重要度）: $1
- 合計: **$1**

## MVPスコープ
- 20主要SaaSテンプレ
- 差分検知
- 影響額の簡易試算
- Discord通知

## マーケ計画
- 「今月の値上げ速報」無料公開
- Indie Hackers / Xで価格改定事例を定期投稿

## 技術スタック
TypeScript, Playwright, Cloudflare Workers, Supabase, Stripe

## リスク
- DOM変更による誤検知
- 対策: 価格領域のみ抽出、手動確認フロー

## 競合分析
- Visualping: 汎用監視でノイズ多
- 差別化: 料金改定特化 + 影響額可視化

## $20達成シナリオ
- Lite $3 x 7人 = $21
- もしくは Pro $7 x 3人 = $21

## ユニットエコノミクス
- ARPU: $4.5
- 変動費: $0.15
- 粗利率: 96.7%
