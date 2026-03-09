# 🧾 GrantWindow Lite

## 概要
自治体・補助金・助成金の公募を追跡し、**自分の業種に合う募集だけを締切順に提示**する保険型ライトSaaS。機会損失を防ぐ。

## 海外事例分析
- 米国ではgrant discoveryツールが中小向けに普及
- 日本は自治体サイト分散で検索負荷が高い
- 「申請期限の見落とし防止」は継続課金と相性が良い

## ターゲット
- 小規模事業者
- 個人事業主
- 補助金申請代行の士業

## 料金
- Free: 1地域・週次通知
- Pro: $8/月（複数地域・即時通知）
- Goal: 3人で$24

## ユーザーフロー
1. 業種/地域登録
2. 募集一覧を適合度順で表示
3. 締切前リマインド
4. 申請チェックリストをDL

## デザインコンセプト
- 行政感を消した「カードUI」
- 締切カウントダウンを強調

## アーキテクチャ
- クローラ: AWS Lambda + EventBridge
- 解析: ルールベース + 軽量LLM補助
- 通知: Email/Discord

## DB設計
- users(id, plan)
- profiles(id, user_id, industry, region)
- grants(id, source, title, deadline, url)
- matches(id, user_id, grant_id, score)
- alerts(id, user_id, grant_id, sent_at)

## コスト見積もり（月）
- Lambda/EventBridge: $1
- DB/Storage: $2
- AI補助: $1
- 合計: $4

## MVPスコープ
- 主要10自治体+中小企業庁
- 適合度スコア
- 期限通知
- 有料化

## マーケ計画
- 地域商工会向け無料デモ
- 士業向け紹介プログラム

## 技術スタック
Python crawler / AWS Lambda / Supabase / OpenAI mini / Stripe

## リスク
- 情報取得の安定性: ソース冗長化と手動修正UIを準備

## 競合分析
- 補助金ポータルは検索中心
- 本サービスは「自分向け+締切駆動」で実行支援

## $20達成シナリオ
- 士業1名経由で顧客3名獲得

## ユニットエコノミクス
- ARPU: $8
- 変動費/人: $0.9
- 粗利率: 88%
