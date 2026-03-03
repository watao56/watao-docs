# 🧾 PayoutProof Lite

## 概要
クリエイター向けに、各プラットフォーム（YouTube/アフィリエイト/ストア）の**入金予定と実入金のズレ**を検知する軽量SaaS。保険型だが、金銭インパクトが直接的な領域に限定。

- カテゴリ: 保険型（1/5）
- 目標: 月$20（$6プラン×4人）

## 海外事例分析
- Visualping系: 差分監視は普遍ニーズ
- クリエイター経済圏: 収益源が分散し、照合作業が手作業
- Gumroad/Patreon文化: 収益可視化ツールの課金耐性あり

示唆: 日本の個人クリエイターは会計知識より「入金漏れ不安」が強い。**監視対象を入金差異に絞る**ことで価値が明確。

## ターゲット
- 複数ASP・広告収益を持つ個人
- 小規模事務所
- 同人/デジタル販売者

## 料金
- Free: 2ソース、月10件照合
- Basic: $6/月（10ソース、毎日照合）
- Pro: $12/月（30ソース、Slack通知、CSV出力）

## ユーザーフロー
1. 収益ソース接続（APIまたはCSV）
2. 入金予定日のルール設定
3. 日次照合で差異を検知
4. 「確認テンプレ文」をワンクリック生成
5. 月次レポートを保存

## デザインコンセプト
- 会計ツールっぽさを減らしたカードUI
- 重要差異のみ赤で強調
- 迷わない3タブ（予定/実績/差異）

## アーキテクチャ
- Next.js + Supabase
- Ingestion: scheduled workers
- 照合ロジック: rule engine (TypeScript)
- 通知: Discord/Email

## DB設計
- payout_sources(id, user_id, source_name, source_type, auth_ref)
- expected_payouts(id, source_id, amount, due_date, currency)
- actual_payouts(id, source_id, amount, paid_at, ref_no)
- mismatches(id, user_id, source_id, diff_type, amount_gap, status)

## コスト見積もり（月）
- インフラ: $0〜$5
- API利用/スクレイプ補助: $0〜$3
- 通知: $0
- 合計: $2〜$8

## MVPスコープ
- CSVインポート中心（API依存を最小化）
- 差異検知ルール3種（未入金/金額不足/遅延）
- Discord通知
- 月次CSV出力

## マーケ計画
- 「入金漏れ発見チェックリスト」無料配布
- クリエイター向け税務コミュニティで紹介
- 実例ベースの短尺動画（漏れ金額の可視化）

## 技術スタック
Next.js / Supabase / Worker Cron / TypeScript Rule Engine / Stripe

## リスク
- 接続先仕様変更
- 金融データ取り扱いへの不安
- 誤検知によるサポート負荷

## 競合分析
- 家計簿系: プラットフォーム収益照合に弱い
- BIツール: 個人には重く高価
- 本案: **入金差異検知だけに絞った軽量保険型**

## $20達成シナリオ
- Basic $6 × 4人 = $24
- もしくは Pro $12 × 2人 = $24

## ユニットエコノミクス
- ARPU: $7.2
- 変動費: $0.8/user
- 粗利: $6.4/user（88.9%）
- 継続要因: 解約すると再び手動照合に戻る痛み
