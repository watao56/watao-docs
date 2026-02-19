# 📦 StockPing — EC在庫切れ・低在庫アラート

## 概要

自社ECサイト（Shopify, BASE, STORES, WooCommerce等）の在庫数を監視し、設定した閾値を下回ったらSlack/メール/LINEでアラートを送るサービス。在庫切れは直接的な機会損失であり、**ECサイトの平均在庫切れ率は8%、年間売上の4.1%が在庫切れによる機会損失**（IHL Group調査）。

## ターゲット

- **メイン**: 個人〜小規模ECショップオーナー（商品数10-500）
- **サブ**: ハンドメイド作家（minne, Creema → 自社ECも運営）
- **拡張**: D2Cブランド、ドロップシッピング事業者

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 1ショップ、50商品、1日1回チェック |
| Pro | $5/月 | 3ショップ、500商品、1時間毎チェック、Slack/LINE通知 |
| Business | $12/月 | 10ショップ、無制限商品、15分毎、需要予測（β） |

## ユーザーフロー

1. **登録**: メールで登録
2. **ショップ連携**: Shopify/BASE/STORES等のAPIキーを入力、またはCSVインポート
3. **閾値設定**: 商品ごとに「在庫X個以下でアラート」を設定（デフォルト: 5個）
4. **通知先設定**: メール / Slack / LINE Notify
5. **監視開始**: APIで在庫数を定期取得
6. **アラート**: 閾値以下→通知。「商品A: 残り3個（閾値5個）」
7. **ダッシュボード**: 在庫推移グラフ、在庫切れ履歴、売れ筋ランキング

## アーキテクチャ

```
[EC Platform APIs]
  Shopify / BASE / STORES / WooCommerce
           ↓
    [Lambda: Stock Poller]
    (EventBridge: 15分〜1日)
           ↓
    [DynamoDB: 在庫データ]
           ↓
    判定: 在庫 < 閾値?
           ↓ Yes
    [通知: SES / Slack / LINE]
           ↓
    [Next.js Dashboard]
```

### コンポーネント

- **Frontend**: Next.js（Vercel無料枠）
- **API**: AWS Lambda + API Gateway
- **DB**: DynamoDB
- **スケジューラ**: EventBridge
- **EC連携**: Shopify Admin API, BASE API, STORES API, WooCommerce REST API
- **通知**: SES + Slack Webhook + LINE Notify
- **認証**: NextAuth.js

## DB設計

### Users テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | UUID |
| email | String | メール |
| plan | String | free/pro/business |

### Shops テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | ユーザーID |
| shopId (SK) | String | ショップID |
| platform | String | shopify/base/stores/woocommerce |
| apiKey | String | APIキー（暗号化） |
| shopUrl | String | ショップURL |

### Products テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| shopId (PK) | String | ショップID |
| productId (SK) | String | 商品ID |
| name | String | 商品名 |
| currentStock | Number | 現在の在庫数 |
| threshold | Number | アラート閾値 |
| lastChecked | Number | 最終チェック時刻 |
| alertSent | Boolean | アラート送信済み |

### StockHistory テーブル (GSI: shopId-date)
| フィールド | 型 | 説明 |
|-----------|------|------|
| productId (PK) | String | 商品ID |
| timestamp (SK) | Number | 記録時刻 |
| stock | Number | 在庫数 |

## コスト見積もり

### インフラコスト（月額・初期）

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
- AI不使用。ルールベースの閾値チェック。

### 100ユーザー時（平均100商品/ユーザー）
| 項目 | 月額 |
|------|------|
| Lambda | $1.50（10,000商品 × 96回/日 × 30日） |
| DynamoDB | $1.00 |
| API Gateway | $1.00 |
| **合計** | **$3.50/月** |

## MVPスコープ

### Phase 1（2週間）
- ユーザー登録/ログイン
- Shopify API連携（在庫取得）
- 閾値設定（商品ごと + 一括）
- 在庫チェックバッチ（EventBridge）
- アラート通知（メール + Slack）
- シンプルダッシュボード（在庫一覧、低在庫ハイライト）

### Phase 2（+1週間）
- BASE, STORES連携
- LINE Notify対応
- 在庫推移グラフ
- CSVインポート/エクスポート
- Stripe決済

### Phase 3（+2週間）
- WooCommerce連携
- 需要予測（過去データからの簡易予測）
- 自動発注リマインド

## マーケ計画

### 初期（1-3ヶ月目）
- **SEO**: 「Shopify 在庫管理 自動化」「EC 在庫切れ 防止」「BASE 在庫アラート」
- **ECコミュニティ**: Shopifyコミュニティ、BASE Creator向けSNS
- **Product Hunt**: 「Stock monitoring for small e-commerce」
- **Twitter/X**: EC運営者向けTips＋ツール紹介

### 中期（3-6ヶ月目）
- **パートナーシップ**: Shopifyアプリストア掲載
- **コンテンツ**: 「在庫切れで年間○万円損している計算」記事
- **紹介プログラム**: ECオーナー同士の口コミ

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 14, TailwindCSS, shadcn/ui, Recharts |
| Backend | AWS Lambda (Node.js 20) |
| DB | DynamoDB |
| Auth | NextAuth.js |
| EC APIs | Shopify Admin API, BASE API, STORES API |
| Hosting | Vercel + AWS |
| CI/CD | GitHub Actions |
| 決済 | Stripe |
| 通知 | SES, Slack Webhook, LINE Notify |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| EC APIのrate limit | チェック頻度制限 | バッチ処理、差分取得 |
| Shopifyアプリストア審査 | ローンチ遅延 | まずはAPI直接連携、後にアプリ化 |
| 大手ECプラットフォーム標準機能化 | 差別化喪失 | マルチプラットフォーム統合が強み |
| 在庫データの正確性 | 誤アラート | Webhook対応で即時反映 |

## 競合分析

| 競合 | 特徴 | StockPingの優位性 |
|------|------|-------------------|
| Shopify標準 | 低在庫メール | 単一プラットフォーム、カスタマイズ不可 |
| Stocky (Shopify) | 在庫管理アプリ | Shopify限定、複雑 |
| TradeGecko | 在庫管理SaaS | $39/月〜。小規模には高い |
| スプレッドシート | 手動管理 | 更新忘れ、リアルタイム性なし |

## $20達成シナリオ

| シナリオ | Pro ($5) | Business ($12) | MRR |
|---------|---------|----------------|-----|
| 最速 | 4人 | 0人 | $20 |
| 現実的（3ヶ月目） | 3人 | 1人 | $27 |
| 保守的（6ヶ月目） | 4人 | 1人 | $32 |

### 達成根拠
- EC市場は年10%成長。小規模EC事業者は増加中
- 在庫切れ1回の機会損失 > 年間の利用料
- Shopify日本ユーザーだけで10万店舗超
- マルチプラットフォーム在庫管理の需要は明確

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0-3（SEO+コミュニティ） |
| ARPU | $6.75（Pro:Business = 3:1想定） |
| 粗利率 | 96.5%（$3.50/$101.25 @100ユーザー） |
| LTV（12ヶ月） | $81.00 |
| LTV/CAC | $27-∞ |
| 月間チャーン率 | 3%（EC運営中は必須ツール） |
| 損益分岐ユーザー数 | 1人 |
