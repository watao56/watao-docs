# 🧾 VATLens Lite

## 概要
海外SaaSの請求メール/明細を取り込み、**VAT・適格請求書要件・円換算管理の抜け漏れ**を検知して通知。確定申告前の「詰み」を防ぐ保険型マイクロSaaS。

## 海外事例分析
- Quaderno/Stripe Tax: 税計算は強いが個人事業主向け導線が重い
- Paddle: B2B請求管理は強いが日本の実務に過不足
- 各種会計SaaS: 海外明細の証憑回収が弱い

## ターゲット
- 海外AI/SaaSを多用する個人開発者
- インボイス対応が必要なフリーランス

## 料金
- Lite: $5/月（50明細）
- Pro: $9/月（300明細、税理士共有）

## ユーザーフロー
1. 転送メール設定
2. AI/OCRで明細抽出
3. 要件不足を色分け表示
4. 月次エクスポート

## デザインコンセプト
「**会計前夜でも怖くないダッシュボード**」。赤黄緑の不足可視化を最優先。

## アーキテクチャ
- Backend: AWS Lambda (Node.js)
- Ingest: SES inbound + S3
- OCR: Textract（不足時のみ）
- DB: DynamoDB
- 通知: Discord/Email

## DB設計
- users(id, plan, tax_profile_json)
- invoices(id, user_id, vendor, amount, currency, issued_at, vat_status, source)
- checks(id, invoice_id, rule_code, severity, message)
- exports(id, user_id, month, csv_url)

## コスト見積もり（月）
- AWS Free Tier後: $2〜$8
- OCR利用: $1〜$4
- AI抽出: $1〜$3
- 合計: **$4〜$12**

## MVPスコープ
- メール転送取込
- 主要10サービス（OpenAI, Anthropic, Figma等）対応
- 不足項目チェック
- CSV出力

## マーケ計画
- 「海外SaaS経費化チェックリスト」配布
- 税理士コミュニティへの紹介連携
- 3月確定申告前に短期キャンペーン

## 技術スタック
AWS Lambda / SES / S3 / DynamoDB / Textract / Stripe

## リスク
- 税務判断の誤解 → “助言ではなく整理支援”を明記
- OCR精度 → 人手修正UIを提供

## 競合分析
会計SaaSは入力後処理中心。VATLens Liteは**証憑回収段階の抜け漏れ防止**に特化。

## $20達成シナリオ
Lite($5)×4人で$20達成。

## ユニットエコノミクス
- ARPU: $5.8（Lite/Pro混在）
- 変動費: $0.9/人
- 粗利: 約84%
