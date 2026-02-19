# 💰 RefundRadar — SaaS SLA違反検知＆返金請求支援

## 概要

利用中のSaaSのステータスページを自動監視し、ダウンタイムがSLA基準を超えた場合に返金額を算出、クレームメールのテンプレートを生成するサービス。多くの企業がSLA違反による返金を請求せず、年間数百〜数千ドルを見逃している。

## ターゲット

- **メイン**: SaaSを5〜20個使っている中小企業・スタートアップのIT担当
- **サブ**: フリーランスエンジニア・デザイナー（AWS, Vercel, Figma等に月$100+支払い）
- **拡張**: 大企業のIT部門（コスト最適化担当）

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 3サービス監視、月次レポート |
| Pro | $5/月 | 15サービス監視、即時通知、クレームテンプレート生成 |
| Business | $15/月 | 50サービス監視、チーム共有、過去データ分析 |

## ユーザーフロー

1. **登録**: メールで登録（10秒）
2. **サービス追加**: 利用中のSaaS名を選択（プリセット100+）またはステータスページURLを入力
3. **SLA設定**: 各サービスのSLA条件を設定（プリセットあり）
4. **監視開始**: 自動でステータスページをポーリング
5. **違反検知**: ダウンタイムがSLA基準を超えたら通知
6. **返金請求**: 返金額の自動算出＋クレームメールテンプレート生成
7. **トラッキング**: 請求の結果（承認/却下）を記録

## アーキテクチャ

```
[ユーザー] → [Next.js SPA] → [API Gateway]
                                    ↓
                              [Lambda Functions]
                                    ↓
                          [DynamoDB] + [SQS]
                                    ↓
                           [Status Page Poller]
                            (EventBridge 5分毎)
                                    ↓
                              [SNS → Email]
```

### コンポーネント

- **Frontend**: Next.js（Vercel無料枠）
- **API**: AWS Lambda + API Gateway
- **DB**: DynamoDB（Free Tier: 25GB, 25 WCU/RCU）
- **キュー**: SQS（Free Tier: 100万リクエスト/月）
- **スケジューラ**: EventBridge（5分間隔ポーリング）
- **通知**: Amazon SES（Free Tier: 62,000通/月）
- **認証**: NextAuth.js（自前実装）

## DB設計

### Users テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | UUID |
| email | String | ログインメール |
| plan | String | free/pro/business |
| createdAt | Number | タイムスタンプ |

### MonitoredServices テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | ユーザーID |
| serviceId (SK) | String | サービス識別子 |
| serviceName | String | サービス名 |
| statusPageUrl | String | ステータスページURL |
| slaUptime | Number | SLA稼働率（例: 99.9） |
| monthlyFee | Number | 月額料金（返金額算出用） |

### Incidents テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| serviceId (PK) | String | サービス識別子 |
| incidentId (SK) | String | インシデントID |
| startedAt | Number | 開始時刻 |
| resolvedAt | Number | 解決時刻 |
| downtimeMinutes | Number | ダウンタイム（分） |
| slaViolation | Boolean | SLA違反かどうか |

### RefundClaims テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | ユーザーID |
| claimId (SK) | String | クレームID |
| serviceId | String | 対象サービス |
| amount | Number | 請求額 |
| status | String | pending/submitted/approved/rejected |

## コスト見積もり

### インフラコスト（月額）

| 項目 | 月額 |
|------|------|
| Vercel（フロントエンド） | $0 |
| Lambda（API + ポーリング） | $0（Free Tier内） |
| DynamoDB | $0（Free Tier内） |
| API Gateway | $0.35（10万リクエスト） |
| EventBridge | $0（Free Tier内） |
| SES | $0（Free Tier内） |
| **合計** | **$0.35/月** |

### AIコスト
- AI不使用。クレームメールはテンプレートベース。

### 100ユーザー時
| 項目 | 月額 |
|------|------|
| Lambda | $0.50 |
| DynamoDB | $0.50 |
| API Gateway | $1.00 |
| SES | $0 |
| **合計** | **$2.00/月** |

## MVPスコープ

### Phase 1（2週間）
- ユーザー登録/ログイン
- 主要SaaS 20サービスのプリセット（AWS, GitHub, Vercel, Slack, Figma等）
- ステータスページ自動ポーリング（5分間隔）
- ダウンタイム記録
- SLA違反検知＆メール通知
- 返金額自動算出
- クレームメールテンプレート生成

### Phase 2（+1週間）
- ダッシュボード（月次稼働率グラフ）
- カスタムステータスページURL対応
- Stripe決済連携
- チーム機能

## マーケ計画

### 初期（1-3ヶ月目）
- **SEO記事**: 「SaaS SLA返金 方法」「AWS SLAクレジット 請求」等
- **Hacker News / Reddit**: 「I built a tool to track SaaS SLA violations」投稿
- **Product Hunt**: ローンチ
- **Twitter/X**: SaaS障害発生時に「このダウンタイムでSLA違反、返金請求できます」と投稿

### 中期（3-6ヶ月目）
- **パートナーシップ**: IT管理ツール（Blissfully, Zylo等）との連携
- **コンテンツマーケ**: 「SaaS SLA返金ガイド」eBook
- **口コミ**: 返金成功事例の共有促進

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 14, TailwindCSS, shadcn/ui |
| Backend | AWS Lambda (Node.js 20) |
| DB | DynamoDB |
| Auth | NextAuth.js |
| Hosting | Vercel (Frontend), AWS (Backend) |
| CI/CD | GitHub Actions |
| 決済 | Stripe |
| メール | Amazon SES |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| ステータスページのフォーマット変更 | ポーリング失敗 | 汎用パーサー＋Statuspage.io API対応 |
| SaaS側がSLA返金を簡素化 | サービスの価値低下 | 監視・レポート機能に価値を移行 |
| 大手参入（Datadog等） | 価格競争 | ニッチ特化（中小向け低価格） |
| Free Tier超過 | コスト増 | ユーザー数に応じて段階的にスケール |

## 競合分析

| 競合 | 特徴 | RefundRadarの優位性 |
|------|------|-------------------|
| StatusGator | ステータスページ集約 | 返金額算出・クレームテンプレートが差別化 |
| Hyperping | 稼働率監視 | SLA特化ではない、返金機能なし |
| Instatus | ステータスページ作成 | 自社用であり、他社監視ではない |
| 手動管理 | ブックマーク | 漏れる、計算しない、面倒 |

## $20達成シナリオ

| シナリオ | Pro ($5) | Business ($15) | MRR |
|---------|---------|----------------|-----|
| 最速 | 4人 | 0人 | $20 |
| 現実的（3ヶ月目） | 3人 | 1人 | $30 |
| 保守的（6ヶ月目） | 2人 | 1人 | $25 |

### 達成根拠
- SaaSを多数利用する企業は増加の一方
- 「返金を見逃している」という認知が広まれば、$5/月は安い保険
- 1社でも年間$50-200の返金を得られれば、$60/年の投資は即回収

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC（顧客獲得コスト） | $0（SEO/コンテンツ主体） |
| ARPU（月額平均単価） | $7.50（Pro:Business = 3:1想定） |
| 粗利率 | 97.3%（$2.00/$75.00 @100ユーザー） |
| LTV（12ヶ月） | $90.00 |
| LTV/CAC | ∞（有機獲得前提） |
| 月間チャーン率 | 5%（保険型で低め） |
| 損益分岐ユーザー数 | 1人（Pro 1人で$5 > $0.35コスト） |
