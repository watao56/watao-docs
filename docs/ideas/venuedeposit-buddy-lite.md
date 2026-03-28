# 🛡️ VenueDeposit Buddy Lite

## 概要
小規模イベント主催者向けに、会場予約の**キャンセル期限・返金条件・入金証跡**を1画面管理する保険寄りマイクロSaaS。失注/返金漏れを防ぐ。

## 海外事例分析
- **HoneyBook / Bonsai**: 契約・請求統合の需要
- **Eventbrite運営者フォーラム**: 会場費トラブル相談が多い
- 日本ギャップ: 会場規約がPDF/メール散在で期限を落としやすい

## ターゲット
- 30〜200人規模の勉強会主催
- コミュニティ運営者
- 小規模イベント制作フリーランス

## 料金
- Free: 2案件
- Lite: **$5/mo**（20案件）
- Pro: $12/mo（無制限＋共同編集）

## ユーザーフロー
1. 会場契約PDFをアップロード
2. AIが返金条件/締切を抽出
3. カレンダー通知を自動生成
4. 入金証憑を添付
5. 締切前リマインド受信

## デザインコンセプト
"**No-deadline-loss**"。タイムライン中心、危険日は赤いピルタグで強調。

## アーキテクチャ
- Front: Next.js
- OCR/抽出: AWS Textract + LLM
- DB: DynamoDB
- File: S3
- Reminder: EventBridge + SES/Discord

## DB設計
- users(id, email, plan)
- venues(id, user_id, name, contact)
- contracts(id, venue_id, file_url, cancel_deadline, refund_rule)
- payments(id, contract_id, amount, status, proof_url)
- reminders(id, contract_id, remind_at, channel, sent)

## コスト見積もり（月）
- AWS基盤: $6
- OCR/LLM: $5
- 通知: $1
- 合計: **$12**

## MVPスコープ
- PDF抽出
- 締切カレンダー
- 3段階リマインド
- 証跡アップロード

## マーケ計画
- connpass主催者向け導線
- テンプレ規約チェックリスト配布
- イベント運営Discordで事例共有

## 技術スタック
Next.js / AWS Textract / Lambda / DynamoDB / S3 / Stripe

## リスク
- OCR誤抽出
- 法的解釈の責任範囲

## 競合分析
HoneyBook等は高機能高価格。Buddy Liteは「締切事故防止」に絞って低価格。

## $20達成シナリオ
- Lite 4ユーザー = $20
- 変動費低く、少人数で目標到達

## ユニットエコノミクス
- ARPU: $5
- 変動費/有料ユーザー: $0.9
- 粗利率: 82%
