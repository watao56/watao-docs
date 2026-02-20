# 🔔 WebhookGuard — Webhook配信監視SaaS

## 概要

Stripe・PayPal・Shopify等の決済/EC系Webhookが正しく配信・処理されているかを24/7監視し、失敗時に即座にアラートを送るSaaS。Webhookの到達遅延・レスポンスエラー・ペイロード異常を検知し、「注文が通ったのに処理されていない」事故を防ぐ。

## 解決する課題

- **Webhook配信失敗に気づかない**: Stripe等はリトライするが、エンドポイントが壊れていると最終的にWebhookが停止される
- **売上損失**: 決済Webhookの未処理=注文確認メール未送信、在庫未更新、出荷未開始
- **サイレント障害**: エラーログを毎日チェックする人はいない。気づくのは顧客クレーム後
- **デプロイ後の破壊**: コード更新でWebhookエンドポイントが壊れるケースが頻発

## ターゲット

| セグメント | ペルソナ | 課題の深刻度 |
|-----------|---------|------------|
| EC事業者 | Shopify/WooCommerceで月商50万円〜 | 🔴 注文漏れ=直接損失 |
| SaaS運営者 | Stripe連携で課金管理 | 🔴 課金失敗=チャーン |
| フリーランス開発者 | 複数クライアントのWebhook管理 | 🟡 信頼喪失リスク |
| Web制作会社 | クライアントサイトの保守 | 🟡 保守品質の証明 |

## 料金プラン

| プラン | 月額 | エンドポイント数 | 監視間隔 | 通知先 |
|--------|------|----------------|---------|--------|
| Free | $0 | 1 | 1時間 | メールのみ |
| Pro | $5/月 | 10 | 5分 | メール+Slack+Discord |
| Business | $15/月 | 50 | 1分 | 全チャネル+PagerDuty |

### $20達成シナリオ

- **最速**: Pro 4人 = $20/月（2ヶ月目）
- **安定**: Pro 3人 + Business 1人 = $30/月（3ヶ月目）
- **必要フリーユーザー数**: 50人（有料転換率8%）

## ユーザーフロー

1. **登録**: メール or GitHub OAuth（10秒）
2. **エンドポイント追加**: WebhookGuardが中継URL発行 → Stripe管理画面に設定
3. **中継方式**: WebhookGuard URL → ユーザーの本来のエンドポイントにプロキシ
4. **監視開始**: レスポンスコード・レイテンシ・ペイロードサイズを記録
5. **アラート**: 失敗時にSlack/メール/Discordで即通知
6. **ダッシュボード**: 直近24h/7d/30dの配信成功率・レイテンシグラフ

### 中継方式の利点
- ユーザーのコード変更不要（URL差し替えのみ）
- 全ペイロードをログ保存（デバッグに便利）
- リトライ機能内蔵（本来のエンドポイントがダウンしても再送）

## アーキテクチャ

```
[Stripe/PayPal/Shopify]
        ↓ Webhook POST
[CloudFront + API Gateway]
        ↓
[Lambda: webhook-receiver]
        ↓ ペイロード保存 + 転送
    ┌───┴───┐
    ↓       ↓
[DynamoDB] [ユーザーの本来のエンドポイント]
    ↓       ↓ レスポンス記録
[Lambda: health-checker] ← EventBridge (5分間隔)
    ↓
[SES/SNS: アラート送信]
```

### コンポーネント

| コンポーネント | 技術 | 役割 |
|--------------|------|------|
| フロントエンド | Next.js (Vercel Free) | ダッシュボード・設定画面 |
| API | API Gateway + Lambda | Webhook受信・転送・API |
| ストレージ | DynamoDB | Webhookログ・ユーザー設定 |
| 監視 | EventBridge + Lambda | 定期ヘルスチェック |
| 通知 | SES + SNS | メール・Slack・Discord通知 |
| 認証 | NextAuth.js | GitHub OAuth + メール |
| 決済 | Stripe | サブスクリプション管理 |

## DB設計

### DynamoDB テーブル

```
Users
  PK: USER#{userId}
  email, plan, stripeCustomerId, createdAt

Endpoints
  PK: USER#{userId}
  SK: EP#{endpointId}
  name, targetUrl, proxyUrl, provider, status, lastSuccess, lastFailure

WebhookLogs
  PK: EP#{endpointId}
  SK: LOG#{timestamp}
  statusCode, latencyMs, payloadSize, retryCount, error
  TTL: 30日（Free）/ 90日（Pro）/ 365日（Business）

Alerts
  PK: USER#{userId}
  SK: ALERT#{timestamp}
  endpointId, type, message, acknowledged
```

## コスト見積もり

### インフラコスト（月額）

| リソース | 無料枠 | 50ユーザー時 | 500ユーザー時 |
|---------|--------|------------|-------------|
| Lambda | 100万リクエスト無料 | $0.00 | $0.50 |
| API Gateway | 100万リクエスト無料 | $0.00 | $1.75 |
| DynamoDB | 25GB+25WCU無料 | $0.00 | $2.00 |
| SES | 62,000通/月無料 | $0.00 | $0.10 |
| CloudFront | 1TB無料 | $0.00 | $0.00 |
| Vercel | Free tier | $0.00 | $0.00 |
| **合計** | | **$0.00** | **$4.35** |

### AI使用: なし
全てルールベース（HTTPステータスコード判定、レイテンシ閾値、パターンマッチ）

## MVPスコープ（2週間）

### Week 1
- [ ] Webhook中継エンドポイント（受信→転送→ログ保存）
- [ ] ユーザー登録・認証
- [ ] エンドポイント追加・管理UI
- [ ] DynamoDBスキーマ・CRUD

### Week 2
- [ ] アラート通知（メール・Slack）
- [ ] ダッシュボード（成功率・レイテンシ表示）
- [ ] Stripe決済連携
- [ ] ランディングページ

### MVP後
- Discord/PagerDuty通知
- 自動リトライ設定
- Webhookペイロード検証ルール
- チーム機能

## マーケティング計画

### Phase 1: ローンチ（1-2週目）
- Product Hunt投稿
- IndieHackers/Reddit/HackerNewsでリリース告知
- 「Stripe Webhook失敗で$X損失した話」系のブログ記事
- Twitter/Xでの事例ツイート

### Phase 2: SEO（3-8週目）
- 「webhook monitoring」「stripe webhook failure」等のキーワード記事
- 「How to debug Stripe webhook failures」チュートリアル
- 各決済プロバイダ別のセットアップガイド

### Phase 3: パートナー（2-3ヶ月目）
- Shopifyアプリストア掲載
- Web制作会社への保守オプション提案
- Stripe公式パートナーディレクトリ

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js 14 (App Router) + Tailwind CSS |
| バックエンド | Node.js + Lambda |
| DB | DynamoDB |
| 認証 | NextAuth.js |
| 決済 | Stripe |
| ホスティング | Vercel (Frontend) + AWS (Backend) |
| CI/CD | GitHub Actions |
| モニタリング | CloudWatch |

## リスク

| リスク | 影響 | 対策 |
|-------|------|------|
| Stripe等が自前監視を強化 | 差別化困難 | マルチプロバイダ対応で差別化 |
| 中継による遅延 | UX悪化 | Edge Lambda使用、P95 < 50ms維持 |
| セキュリティ（ペイロード保存） | 信頼性問題 | AES-256暗号化、SOC2準拠目標 |
| 大量トラフィック時のコスト | 利益圧迫 | エンドポイント数制限、従量課金検討 |

## 競合分析

| サービス | 月額 | 弱点 |
|---------|------|------|
| Hookdeck | $25〜 | 高い。$20目標層には過剰 |
| Svix | $50〜 | Webhook送信側向け。受信監視ではない |
| RequestBin | 無料 | デバッグ用。監視・アラートなし |
| 自前ログ監視 | $0 | 設定が面倒、見逃しがち |

### WebhookGuardの優位性
- **$5/月から**: 競合の1/5〜1/10の価格
- **受信側特化**: 送信側ツール（Svix）とは逆のポジション
- **中継方式**: コード変更不要、1分で設定完了
- **アラート特化**: ログ閲覧ではなく「異常時に教える」

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC（獲得コスト） | $0（SEO+コミュニティ） |
| ARPU | $5〜$15/月 |
| インフラコスト/ユーザー | $0.01〜$0.09/月 |
| 粗利率 | **99.2%**（50ユーザー時） |
| LTV（12ヶ月継続） | $60〜$180 |
| LTV/CAC | ∞（有料マーケなし） |
| 月次チャーン想定 | 5%（保険型のため低い） |
| $20達成必要人数 | **Pro 4人** |
