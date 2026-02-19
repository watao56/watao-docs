# 📬 FormLeadPing — フォーム問い合わせ対応漏れ防止

## 概要

お問い合わせフォーム（Googleフォーム、Typeform、Contact Form 7、自作フォーム等）からの送信をWebhookで受信し、対応状況を管理。未対応のまま一定時間が経過したらアラートを送るサービス。**Gmail API不要**。フォームのWebhook/メール転送を設定するだけで、問い合わせの対応漏れを完全に防止する。

CRMのように複雑な機能は不要。「問い合わせが来た→対応した/してない」だけをシンプルに管理する。

## ターゲット

- **メイン**: ホームページからの問い合わせで商売している事業者（Web制作会社、税理士事務所、工務店、塾）
- **サブ**: フリーランス（ポートフォリオサイトの問い合わせ）
- **拡張**: ECサイト運営者（カスタマーサポートのメール対応）

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 1フォーム、月30件、4時間ルール、メール通知 |
| Pro | $4/月 | 5フォーム、月300件、カスタムルール、Slack/LINE通知 |
| Business | $10/月 | 20フォーム、無制限、チーム管理、対応時間レポート |

## ユーザーフロー

1. **登録**: メールで登録
2. **Webhook URL発行**: FormLeadPingがユニークなWebhook URLを発行
3. **フォーム連携**: フォームのWebhook送信先にURLを設定（Googleフォーム: GAS、Typeform: Webhooks、CF7: WP Mail SMTP転送）
4. **受信開始**: フォーム送信がFormLeadPingに届く
5. **未対応チェック**: 設定時間経過後も「対応済み」マークがなければアラート送信
6. **対応記録**: メール内のワンクリックリンクまたはダッシュボードで「対応済み」をマーク
7. **レポート**: 平均対応時間、未対応率の推移

## アーキテクチャ

```
[各種フォーム]
  Google Forms → GAS → Webhook
  Typeform → Webhook
  CF7 → WP Mail SMTP → Email転送
  自作フォーム → Webhook POST
           ↓
[API Gateway: Webhook受信エンドポイント]
           ↓
[Lambda: 受信処理]
           ↓
[DynamoDB: リード管理]
           ↓
[EventBridge: 15分毎チェック]
           ↓
[Lambda: 未対応チェッカー]
           ↓
判定: 未対応 > 閾値?
           ↓ Yes
[SES / Slack / LINE Notify]
```

### コンポーネント

- **Frontend**: Next.js（Vercel無料枠）
- **API**: AWS Lambda + API Gateway
- **DB**: DynamoDB
- **Webhook受信**: API Gateway（POSTエンドポイント）
- **メール受信**: SES Inbound（メール転送対応）
- **スケジューラ**: EventBridge（15分間隔）
- **通知**: SES + Slack Webhook + LINE Notify
- **認証**: NextAuth.js

## DB設計

### Users テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | UUID |
| email | String | メール |
| plan | String | free/pro/business |

### Forms テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | ユーザーID |
| formId (SK) | String | フォームID |
| name | String | フォーム名（「お問い合わせ」等） |
| webhookUrl | String | 発行したWebhook URL |
| alertMinutes | Number | 未対応アラート閾値（分） |
| notifyChannels | List | 通知先 |

### Leads テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| formId (PK) | String | フォームID |
| leadId (SK) | String | リードID |
| receivedAt | Number | 受信日時 |
| data | Map | フォーム送信データ（名前、メール、内容等） |
| status | String | new/in-progress/done/ignored |
| respondedAt | Number | 対応完了日時（null=未対応） |
| assignee | String | 担当者（Business向け） |
| alertSent | Boolean | アラート送信済み |

### ResponseMetrics テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| formId (PK) | String | フォームID |
| date (SK) | String | 日付 |
| totalLeads | Number | 受信数 |
| avgResponseMin | Number | 平均対応時間 |
| unreplied | Number | 未対応数 |

## コスト見積もり

### インフラコスト（月額）

| 項目 | 月額 |
|------|------|
| Vercel | $0 |
| Lambda | $0（Free Tier内） |
| DynamoDB | $0（Free Tier内） |
| API Gateway | $0.15 |
| EventBridge | $0 |
| SES | $0 |
| **合計** | **$0.15/月** |

### AIコスト
- AI不使用。全てルールベース。

### 200ユーザー時
| 項目 | 月額 |
|------|------|
| Lambda | $0.50 |
| DynamoDB | $0.50 |
| API Gateway | $1.50 |
| SES | $0 |
| **合計** | **$2.50/月** |

## MVPスコープ

### Phase 1（10日）
- ユーザー登録/ログイン
- フォーム作成＋Webhook URL発行
- Webhook受信→DynamoDBに保存
- 未対応チェックバッチ
- メール通知（未対応アラート）
- ダッシュボード（リード一覧、ステータス管理）
- メール内ワンクリック「対応済み」マーク

### Phase 2（+5日）
- Slack / LINE Notify対応
- Stripe決済
- Googleフォーム連携ガイド（GASテンプレート）
- 対応時間レポート

### Phase 3（+1週間）
- SES Inbound（メール転送受信）
- チーム機能（担当者アサイン）
- Zapier/Make連携
- 月次レポートPDF

## マーケ計画

### 初期（1-3ヶ月目）
- **SEO**: 「問い合わせ フォーム 管理」「お問い合わせ 返信忘れ 防止」「フォーム送信 通知」
- **Web制作会社向け**: 「納品サイトに導入→クライアントに月額課金」のビジネスモデル提案
- **WordPress界隈**: Contact Form 7ユーザー向けHow-to記事
- **Product Hunt**: 「Never miss a lead from your contact form」

### 中期（3-6ヶ月目）
- **パートナーシップ**: Web制作会社に導入→クライアント紹介
- **Googleフォーム公式Add-on**: Marketplace掲載
- **事例**: 「問い合わせ対応時間を4時間→30分に短縮した事例」

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 14, TailwindCSS, shadcn/ui |
| Backend | AWS Lambda (Node.js 20) |
| DB | DynamoDB |
| Auth | NextAuth.js |
| Webhook | API Gateway (POST endpoint) |
| Email Inbound | SES Inbound (Phase 3) |
| Hosting | Vercel + AWS |
| CI/CD | GitHub Actions |
| 決済 | Stripe |
| 通知 | SES, Slack Webhook, LINE Notify |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| Webhook設定のハードル | 非エンジニアに難しい | 各フォームサービス別の設定ガイド動画を用意 |
| 「対応済み」マークの手間 | 使われなくなる | メール内ワンクリック。Slack botからもマーク可能 |
| CRMとの比較 | 「HubSpot無料版でいい」 | CRMは複雑すぎる。FormLeadPingは「対応した/してない」だけ |
| フォームスパム | 偽リード大量受信 | reCAPTCHA推奨、スパムフィルタ（メールアドレスパターン） |

## 競合分析

| 競合 | 特徴 | FormLeadPingの優位性 |
|------|------|---------------------|
| HubSpot Free CRM | 無料CRM | 機能過多で設定が複雑。フォーム対応追跡に特化していない |
| Notion/スプレッドシート | 手動管理 | 自動受信なし、アラートなし |
| Zapier + Slack | 通知は可能 | 「対応済み管理」がない。通知だけで追跡なし |
| FormShield（自社） | フォーム死活監視 | FormShieldは「フォームが壊れていないか」、FormLeadPingは「返信したか」。補完関係 |

## $20達成シナリオ

| シナリオ | Pro ($4) | Business ($10) | MRR |
|---------|---------|----------------|-----|
| 最速 | 5人 | 0人 | $20 |
| 現実的（3ヶ月目） | 4人 | 1人 | $26 |
| 保守的（6ヶ月目） | 5人 | 2人 | $40 |

### 達成根拠
- 「問い合わせが来ているのに返信を忘れた」は全事業者の共通痛み
- 1件の失注損失（$100-10,000）>>年間利用料$48
- Web制作会社が顧客に導入推薦→1社で5-10サイト分のアカウント
- FormShieldとのクロスセル可能

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0-3（SEO+Web制作会社パートナー） |
| ARPU | $5.50（Pro:Business = 3:1想定） |
| 粗利率 | 98.5%（$0.15/$27.50 @5ユーザー） |
| LTV（12ヶ月） | $66.00 |
| LTV/CAC | $22-∞ |
| 月間チャーン率 | 4%（問い合わせが来る限り必要） |
| 損益分岐ユーザー数 | 1人 |
