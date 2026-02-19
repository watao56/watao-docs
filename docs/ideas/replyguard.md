# 📩 ReplyGuard — 顧客問い合わせ未返信アラート

## 概要

ビジネスメール（Gmail / Google Workspace）を監視し、顧客からの問い合わせに一定時間返信していない場合にSlack/メール/LINEでアラートを送るサービス。問い合わせ対応の遅延は直接的な失注につながり、**返信が1時間遅れるごとにコンバージョン率が7%低下する**（Harvard Business Review調査）。

## ターゲット

- **メイン**: 1-10人規模の中小企業・個人事業主（問い合わせ管理が属人的）
- **サブ**: フリーランス（見積もり依頼を見逃しがち）
- **拡張**: 不動産・保険・教育等の反響営業が重要な業界

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 1メールアカウント、4時間ルール、1日1回レポート |
| Pro | $5/月 | 3アカウント、カスタムルール、即時通知、Slack連携 |
| Team | $12/月 | 10アカウント、チームダッシュボード、エスカレーション |

## ユーザーフロー

1. **登録**: Googleアカウントで登録（OAuth）
2. **ルール設定**: 「未返信X時間でアラート」を設定（デフォルト: 2時間）
3. **フィルタ設定**: 対象ラベル/送信者ドメイン除外（ニュースレター等）
4. **通知先設定**: メール / Slack / LINE Notify
5. **監視開始**: Gmail APIで受信メールを定期チェック
6. **アラート**: 未返信メールを検知→通知送信（件名、送信者、経過時間）
7. **ダッシュボード**: 平均返信時間、未返信件数の推移

## アーキテクチャ

```
[Gmail] ← Gmail API (Pub/Sub Push)
           ↓
    [Cloud Functions / Lambda]
           ↓
    [DynamoDB: メール状態管理]
           ↓
    [チェッカー (EventBridge 15分毎)]
           ↓
    判定: 未返信 > 閾値?
           ↓ Yes
    [通知送信: SES / Slack Webhook / LINE Notify]
```

### コンポーネント

- **Frontend**: Next.js（Vercel無料枠）
- **API**: AWS Lambda + API Gateway
- **DB**: DynamoDB
- **メール監視**: Gmail API（Pub/Sub push通知 or ポーリング）
- **スケジューラ**: EventBridge（15分間隔チェック）
- **通知**: SES + Slack Incoming Webhook + LINE Notify API
- **認証**: NextAuth.js + Google OAuth

## DB設計

### Users テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | UUID |
| email | String | メインメール |
| plan | String | free/pro/team |
| slackWebhook | String | Slack通知先 |
| lineToken | String | LINE Notify トークン |
| alertThresholdMin | Number | 未返信アラート閾値（分） |

### EmailAccounts テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | ユーザーID |
| accountId (SK) | String | Gmailアカウント識別子 |
| accessToken | String | OAuth token (暗号化) |
| refreshToken | String | Refresh token (暗号化) |
| filters | Map | 除外ルール |

### TrackedEmails テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| accountId (PK) | String | アカウントID |
| threadId (SK) | String | Gmailスレッド |
| from | String | 送信者 |
| subject | String | 件名 |
| receivedAt | Number | 受信時刻 |
| repliedAt | Number | 返信時刻（null=未返信） |
| alertSent | Boolean | アラート送信済みか |

### ResponseMetrics テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| accountId (PK) | String | アカウントID |
| date (SK) | String | 日付 |
| avgResponseMin | Number | 平均返信時間（分） |
| totalReceived | Number | 受信数 |
| totalUnreplied | Number | 未返信数 |

## コスト見積もり

### インフラコスト（月額・初期）

| 項目 | 月額 |
|------|------|
| Vercel | $0 |
| Lambda | $0（Free Tier内） |
| DynamoDB | $0（Free Tier内） |
| API Gateway | $0.20 |
| EventBridge | $0 |
| SES | $0 |
| **合計** | **$0.20/月** |

### AIコスト
- AI不使用。ルールベースの未返信検知。

### 100ユーザー時
| 項目 | 月額 |
|------|------|
| Lambda | $1.00 |
| DynamoDB | $0.50 |
| API Gateway | $1.50 |
| SES | $0 |
| **合計** | **$3.00/月** |

## MVPスコープ

### Phase 1（2週間）
- Google OAuthログイン
- Gmail API連携（受信メール取得）
- 未返信メール検知ロジック（スレッドベース）
- アラート通知（メール + Slack Webhook）
- シンプルダッシュボード（未返信一覧、返信時間推移）
- フィルタ設定（除外ドメイン、ラベル）

### Phase 2（+1週間）
- LINE Notify対応
- チーム機能（複数アカウント統合ダッシュボード）
- Stripe決済
- 日次/週次サマリーメール

## マーケ計画

### 初期（1-3ヶ月目）
- **SEO**: 「問い合わせ 返信忘れ 防止」「Gmail 未返信 アラート」
- **日本市場特化**: 「顧客対応 レスポンス 改善」で日本語記事
- **Product Hunt**: 英語圏向けローンチ
- **Twitter/X**: 「問い合わせ1時間以内に返信しないと78%の顧客は競合に流れる」等のファクト投稿

### 中期（3-6ヶ月目）
- **パートナーシップ**: 中小企業向けSaaS（freee, マネーフォワード）コミュニティ
- **紹介プログラム**: 既存ユーザーの紹介で1ヶ月無料
- **業界特化LP**: 不動産、保険、教育向けLP

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 14, TailwindCSS, shadcn/ui |
| Backend | AWS Lambda (Node.js 20) |
| DB | DynamoDB |
| Auth | NextAuth.js + Google OAuth |
| Email API | Gmail API (googleapis) |
| Hosting | Vercel + AWS |
| CI/CD | GitHub Actions |
| 決済 | Stripe |
| 通知 | SES, Slack Webhook, LINE Notify |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| Google OAuth審査 | ローンチ遅延 | 早期申請、限定公開スコープで開始 |
| Gmail API rate limit | 監視遅延 | Pub/Sub push通知で効率化 |
| プライバシー懸念 | 登録躊躇 | メール本文は読まない（メタデータのみ）、SOC2準拠方針 |
| Gmailのみ対応 | 市場限定 | Phase 2でOutlook対応 |

## 競合分析

| 競合 | 特徴 | ReplyGuardの優位性 |
|------|------|-------------------|
| Zendesk | 大企業向けCS管理 | $49/月〜。中小には高すぎる |
| Freshdesk | チケット管理 | 問い合わせ管理全体。ReplyGuardは「未返信検知」に特化で軽量 |
| Gmail標準 | スヌーズ/リマインダー | 手動設定。自動検知がない |
| Front | 共有受信箱 | $19/人/月〜。高い |

## $20達成シナリオ

| シナリオ | Pro ($5) | Team ($12) | MRR |
|---------|---------|------------|-----|
| 最速 | 4人 | 0人 | $20 |
| 現実的（3ヶ月目） | 3人 | 1人 | $27 |
| 保守的（6ヶ月目） | 2人 | 2人 | $34 |

### 達成根拠
- 問い合わせ対応は全ビジネスの共通課題
- 1件の失注が$100-1000の損失→$5/月は即元が取れる
- Gmail連携の手軽さで無料→有料転換しやすい

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0（SEO/コンテンツ主体） |
| ARPU | $6.75（Pro:Team = 3:1想定） |
| 粗利率 | 98.5%（$3.00/$202.50 @30ユーザー） |
| LTV（12ヶ月） | $81.00 |
| LTV/CAC | ∞（有機獲得前提） |
| 月間チャーン率 | 4%（業務必須ツール化しやすい） |
| 損益分岐ユーザー数 | 1人 |
