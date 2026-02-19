# 🔒 SSLWatcher — マルチドメインSSL証明書一括監視

## 概要

複数ドメインのSSL証明書の有効期限、設定ミス（中間証明書の欠落、弱い暗号スイート、証明書チェーン不備）を一括監視し、問題があればアラートを送るサービス。SSL証明書の期限切れは**ブラウザに「この接続は安全ではありません」と表示**→ユーザーが離脱→売上直撃。Let's Encrypt自動更新の失敗、ワイルドカード証明書の管理漏れ、CDN側の設定ミスなど、「自動更新しているつもりが実は失敗していた」ケースが多発。

**CertRemindとの違い**: CertRemindは個人の資格・免許（運転免許、医師免許等）の更新リマインダー。SSLWatcherはWebサイトのSSL証明書の**技術的な監視**（有効期限だけでなく、設定の正常性チェックを含む）。

## ターゲット

- **メイン**: Web制作会社（クライアントサイト10-100ドメインを管理）
- **サブ**: 複数サービスを運営するスタートアップ
- **拡張**: SIer・MSP（マネージドサービスプロバイダー）

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 3ドメイン、有効期限チェックのみ、メール通知 |
| Pro | $4/月 | 30ドメイン、設定チェック（中間証明書、暗号スイート等）、Slack/LINE通知 |
| Agency | $12/月 | 200ドメイン、クライアント別管理、月次レポートPDF、API |

## ユーザーフロー

1. **登録**: メールで登録
2. **ドメイン追加**: ドメインを入力（一括ペースト対応）
3. **初回スキャン**: SSL証明書の有効期限、発行者、暗号スイート、証明書チェーンを取得
4. **結果表示**: ヘルスステータス（✅ 正常 / ⚠️ 期限間近 / ❌ 問題あり）
5. **監視開始**: 毎日自動チェック
6. **アラート**: 期限30日前、設定異常検知時に通知
7. **レポート**: 月次でクライアント別にPDF生成（Agency向け）

## アーキテクチャ

```
[EventBridge: 毎日 3:00 AM]
         ↓
[Lambda: SSL Checker]
(Node.js tls.connect でSSL情報取得)
         ↓
チェック項目:
- 有効期限
- 証明書チェーン完全性
- 中間証明書の存在
- 暗号スイートの強度
- HSTS設定
- OCSP Stapling
         ↓
[DynamoDB: 結果保存]
         ↓
判定: 問題あり?
         ↓ Yes
[SES / Slack / LINE]
         ↓
[Next.js Dashboard]
```

### コンポーネント

- **Frontend**: Next.js（Vercel無料枠）
- **API**: AWS Lambda + API Gateway
- **DB**: DynamoDB
- **SSLチェック**: Node.js `tls` モジュール（外部API不要）
- **スケジューラ**: EventBridge
- **通知**: SES + Slack Webhook + LINE Notify
- **認証**: NextAuth.js

## DB設計

### Users テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | UUID |
| email | String | メール |
| plan | String | free/pro/agency |

### Domains テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | ユーザーID |
| domainId (SK) | String | ドメインID |
| domain | String | ドメイン名 |
| clientName | String | クライアント名（Agency向け） |
| lastCheck | Number | 最終チェック日時 |
| status | String | healthy/warning/critical |

### SSLChecks テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| domainId (PK) | String | ドメインID |
| checkDate (SK) | String | チェック日 |
| expiresAt | String | 証明書有効期限 |
| issuer | String | 発行者（Let's Encrypt等） |
| daysRemaining | Number | 残り日数 |
| chainValid | Boolean | 証明書チェーン正常 |
| intermediatePresent | Boolean | 中間証明書あり |
| cipherStrength | String | 暗号スイート強度 |
| hstsEnabled | Boolean | HSTS設定 |
| ocspStapling | Boolean | OCSP Stapling |
| overallScore | String | A-F評価 |
| issues | List | 検知された問題 |

## コスト見積もり

### インフラコスト（月額）

| 項目 | 月額 |
|------|------|
| Vercel | $0 |
| Lambda | $0（Free Tier内） |
| DynamoDB | $0（Free Tier内） |
| API Gateway | $0.10 |
| EventBridge | $0 |
| SES | $0 |
| **合計** | **$0.10/月** |

### 外部APIコスト
- **$0**: SSL情報はNode.js `tls.connect`で直接取得。外部API不要。

### AIコスト
- AI不使用。全てルールベースのチェック。

### 300ユーザー時（平均20ドメイン/ユーザー）
| 項目 | 月額 |
|------|------|
| Lambda | $1.80（6000ドメイン × 30日） |
| DynamoDB | $0.50 |
| API Gateway | $0.80 |
| **合計** | **$3.10/月** |

## MVPスコープ

### Phase 1（10日）
- ユーザー登録/ログイン
- ドメイン追加（一括対応）
- SSL証明書チェック（有効期限、チェーン、中間証明書）
- ヘルスダッシュボード（ステータス一覧）
- メールアラート（期限30日前、問題検知時）

### Phase 2（+5日）
- 暗号スイート・HSTS・OCSPチェック
- Slack / LINE通知
- Stripe決済
- スコアリング（A-F評価）

### Phase 3（+1週間）
- Agency向けクライアント管理
- 月次レポートPDF
- API提供（外部ツール連携）
- ドメイン一括インポート（CSV）

## マーケ計画

### 初期（1-3ヶ月目）
- **SEO**: 「SSL証明書 期限切れ チェック」「SSL 設定 確認」「中間証明書 確認方法」
- **Web制作コミュニティ**: 「クライアントサイトのSSL、ちゃんと管理できてますか？」
- **Product Hunt**: 「SSL monitoring for agencies managing 100+ domains」
- **Twitter/X**: SSL証明書切れの事例（大企業の証明書切れニュース時に投稿）

### 中期（3-6ヶ月目）
- **Web制作会社パートナー**: 「クライアントへの保守サービスに組み込む」提案
- **SSL Labsテスト結果との比較**: 「毎回手動でSSL Labsに打ち込んでませんか？」
- **セキュリティ監査対応**: 証明書管理の監査証跡として

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 14, TailwindCSS, shadcn/ui |
| Backend | AWS Lambda (Node.js 20) |
| DB | DynamoDB |
| SSL Check | Node.js tls module (ネイティブ) |
| Auth | NextAuth.js |
| Hosting | Vercel + AWS |
| CI/CD | GitHub Actions |
| 決済 | Stripe |
| 通知 | SES, Slack Webhook, LINE Notify |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| Lambda→外部SSL接続のタイムアウト | チェック失敗 | タイムアウト10秒、リトライ3回 |
| SSL Labs等の無料ツール | 差別化困難 | SSL Labsは手動・1ドメインずつ。一括管理+自動アラートが差別化 |
| Let's Encrypt自動更新の普及 | 需要減 | 「自動更新が失敗している」検知が本来の価値 |
| ファイアウォールでブロック | チェック不能 | 標準的なTLS接続なのでほぼ問題なし |

## 競合分析

| 競合 | 特徴 | SSLWatcherの優位性 |
|------|------|-------------------|
| SSL Labs | 無料、手動テスト | 1ドメインずつ手動。自動監視なし |
| UptimeRobot (SSL監視) | 無料プランあり | SSL設定の詳細チェックはない（期限のみ） |
| Keychest | SSL証明書監視 | $19/月〜。Agencyには高い |
| certbot --dry-run | サーバー上で実行 | サーバーアクセス不要。外部からのチェック |

## $20達成シナリオ

| シナリオ | Pro ($4) | Agency ($12) | MRR |
|---------|---------|--------------|-----|
| 最速 | 2人 | 1人 | $20 |
| 現実的（3ヶ月目） | 3人 | 2人 | $36 |
| 保守的（6ヶ月目） | 5人 | 3人 | $56 |

### 達成根拠
- Web制作会社1社がAgencyプランを導入すれば、一気に$12
- SSL証明書切れ→「安全でない接続」表示は即座に問い合わせ殺到→クライアントからの信頼失墜
- 「毎日手動でSSL Labsを30ドメイン分チェックしている」人は$4/月で即決
- Web制作会社の保守契約に「SSL監視付き」として組み込みやすい

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0-2（SEO+Web制作コミュニティ） |
| ARPU | $6.67（Pro:Agency = 2:1想定） |
| 粗利率 | 99.5%（$0.10/$33.35 @5ユーザー） |
| LTV（12ヶ月） | $80.00 |
| LTV/CAC | $40-∞ |
| 月間チャーン率 | 3%（ドメイン管理中は解約理由なし） |
| 損益分岐ユーザー数 | 1人 |
