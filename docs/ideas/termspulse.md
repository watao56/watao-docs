# 🧾 TermsPulse

## 概要
利用規約変更の“重要差分だけ”を、フリーランス向けに日本語で3行要約するミニ保険型SaaS。保険型は本バッチでは1案のみに限定。

## 海外事例分析
- Terms of Service; Didn’t Read: 啓発中心
- Termly/Iubenda: 企業向けコンプラ管理
- 日本ギャップ: 個人事業主が「どこが危ないか」を即判断できる体験が不足

## ターゲット
- ノーコード制作者
- EC個人運営
- 複数SaaSを使うフリーランス

## 料金
- Free: 3サービスまで監視
- Pro: $7/月（20サービス、緊急通知）

## ユーザーフロー
1. 監視対象SaaSを選択
2. 変更検知時に差分抽出
3. AIがリスクレベル付き要約
4. 必要ならテンプレ対応策を提示

## デザインコンセプト
- 「リーガル信号機」
- 赤黄緑で視認性重視、本文は短く

## アーキテクチャ
- クローラ: Lambda + EventBridge
- 差分: text diff
- 要約: gpt-4o-mini
- 通知: Discord/Email

## DB設計
- users(id, email, plan)
- watch_targets(id, user_id, service_name, url)
- snapshots(id, target_id, hash, content, fetched_at)
- diffs(id, target_id, severity, summary, actions)
- notifications(id, user_id, diff_id, channel, sent_at)

## コスト見積もり（月）
- Lambda/EventBridge: $1
- DB/Storage: $1
- AI要約: $2
- 合計: **約$4**

## MVPスコープ
- 主要30サービスの監視
- 差分要約
- Discord通知

## マーケ計画
- 「今週の規約変更ダイジェスト」を無料公開
- フリーランスコミュニティで検証導入
- 重要変更の実例ベース訴求

## 技術スタック
Node.js / AWS Lambda / DynamoDB / S3 / OpenAI mini

## リスク
- 法的助言と誤解される可能性
- 変更検知漏れリスク

## 競合分析
- ToSDRより個人運用向け
- Termlyより軽量・低価格

## $20達成シナリオ
- Pro 3人で $21 MRR

## ユニットエコノミクス
- ARPU: $7
- 変動費/人: $0.6
- 粗利率: 91%
