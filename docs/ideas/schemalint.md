# 📐 SchemaLint — 構造化データ破損検知SaaS

## 概要

WebサイトのJSON-LD/Microdata等の構造化データ（Schema.org）を定期監視し、破損・消失・エラーを検知してアラートを送るSaaS。「デプロイで構造化データが壊れてリッチスニペットが消えた」「商品ページのレビュー星が検索結果から消失」「パンくずリストが出なくなった」といったSEO資産の損失を自動検知。

## 解決する課題

- **リッチスニペット消失に気づかない**: 構造化データが壊れてもサイト表示は正常。検索結果からのみ消える
- **CTR低下=売上直撃**: リッチスニペット（星評価、価格、FAQ等）がある vs ないでCTRが最大30%差
- **デプロイによる破壊**: テンプレート更新、CMS変更、プラグイン更新でJSON-LDが壊れる
- **Google Search Consoleの遅延**: 構造化データエラーの反映に数日〜数週間かかる
- **手動チェックの限界**: Rich Results Testは1ページずつ手動。100ページのサイトは現実的に不可能

### 被害額の試算
- ECサイト：リッチスニペット消失でCTR 20%低下 → 月間検索流入1万クリックのサイトで2,000クリック損失 → CPC ¥100換算で月20万円相当

## ターゲット

| セグメント | ペルソナ | 課題の深刻度 |
|-----------|---------|------------|
| ECサイト運営者 | 商品の星評価・価格表示が命 | 🔴 リッチスニペット消失=CTR激減 |
| メディア/ブログ | FAQスニペット・記事表示 | 🔴 検索露出低下=PV減 |
| Web制作会社 | SEO施策の品質保証 | 🟡 構造化データ実装の保守 |
| レストラン/店舗 | Googleビジネス連携 | 🟡 営業時間・レビュー表示 |

## 料金プラン

| プラン | 月額 | ページ数 | 監視間隔 | 機能 |
|--------|------|---------|---------|------|
| Free | $0 | 10ページ | 週1回 | 基本チェック+メール |
| Pro | $5/月 | 100ページ | 日1回 | 全Schema対応+Slack+差分 |
| Agency | $12/月 | 1000ページ（複数サイト） | 6時間 | PDF報告+クライアント管理 |

### $20達成シナリオ

- **最速**: Pro 4人 = $20/月（2ヶ月目）
- **安定**: Pro 2人 + Agency 1人 = $22/月（3ヶ月目）
- **必要フリーユーザー数**: 40人（有料転換率10%）

## ユーザーフロー

1. **登録**: メール or Google OAuth
2. **サイト追加**: URLを入力 → 自動クロールでページ一覧取得
3. **初回スキャン**: 各ページのJSON-LD/Microdataを抽出・バリデーション（1-3分）
4. **結果表示**: ページごとのSchema種別一覧（Product, FAQPage, Article, BreadcrumbList等）
5. **ステータス**: ✅ Valid / ⚠️ Warning / ❌ Error / 🚫 Missing
6. **定期監視**: スケジュールに従い自動チェック
7. **アラート**: エラー発生 or Schema消失時に即通知
8. **差分表示**: 前回と今回のJSON-LD diff（何が変わった/壊れたか一目瞭然）

## アーキテクチャ

```
[EventBridge: スケジューラ]
        ↓
[Lambda: crawler]
    └── ページ一覧取得（sitemap.xml or リンク辿り）
        ↓
[SQS: ページキュー]
        ↓
[Lambda: schema-scanner] (並列実行)
    ├── HTML取得
    ├── JSON-LD / Microdata 抽出
    ├── Schema.org バリデーション
    ├── 前回結果と差分比較
    └── → DynamoDB (結果保存)
        ↓ エラー/消失検知時
[Lambda: notifier]
    └── SES / Slack / Discord

[Next.js Dashboard (Vercel)]
    └── API Gateway → Lambda → DynamoDB
```

## DB設計

### DynamoDB テーブル

```
Users
  PK: USER#{userId}
  email, plan, stripeCustomerId

Sites
  PK: USER#{userId}
  SK: SITE#{siteId}
  url, pageCount, lastScan, schemaHealth (healthy/warning/error)

Pages
  PK: SITE#{siteId}
  SK: PAGE#{pageUrlHash}
  url, schemas: [{ type, status, lastValid }], lastScan

ScanResults
  PK: SITE#{siteId}
  SK: SCAN#{timestamp}
  totalPages, validSchemas, warnings, errors, missingSchemas, newIssues

SchemaSnapshots
  PK: PAGE#{pageUrlHash}
  SK: SNAP#{timestamp}
  schemas: [{ type, jsonLd, status, errors }]
  TTL: 30日(Free) / 365日(Pro/Agency)
```

## コスト見積もり

| リソース | 50ユーザー時 | 500ユーザー時 |
|---------|------------|-------------|
| Lambda（スキャン） | $0.00 | $2.00 |
| SQS | $0.00 | $0.40 |
| DynamoDB | $0.00 | $2.00 |
| SES | $0.00 | $0.05 |
| Vercel | $0.00 | $0.00 |
| **合計** | **$0.00** | **$4.45** |

### AI使用: なし
JSON-LDパース + Schema.orgバリデーション（schema-dts / ajv）のみ。ルールベース。

## MVPスコープ（2週間）

### Week 1
- [ ] HTML取得 + JSON-LD/Microdata抽出Lambda
- [ ] Schema.orgバリデーション（必須プロパティチェック）
- [ ] ユーザー登録・サイト追加UI
- [ ] スキャン結果一覧表示

### Week 2
- [ ] 定期監視（EventBridge + SQS）
- [ ] 差分検知・アラート通知
- [ ] Schema種別ごとのステータスダッシュボード
- [ ] Stripe決済・LP

### MVP後
- Google Rich Results Test API連携
- Search Console連携（実際のリッチスニペット表示状況）
- 修正コード自動生成
- WordPress プラグイン

## マーケティング計画

### Phase 1: 無料ツール（1-2週目）
- 「あなたのサイトの構造化データ、壊れてない？」無料1回チェックツール
- Product Hunt・IndieHackers投稿
- SEO系Twitterでの事例ツイート

### Phase 2: SEO（1-3ヶ月目）
- 「JSON-LD 書き方」「構造化データ テスト」「リッチスニペット 消えた」キーワード
- 「構造化データが壊れてCTRが30%下がった話」記事
- Schema種別ごとの実装ガイド（Product, FAQ, HowTo, Recipe等）

### Phase 3: 制作会社（3-6ヶ月目）
- Web制作会社のSEO保守メニューに提案
- CMS（WordPress/Shopify）プラグイン展開
- SEOコンサルタントとのアフィリエイト

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js 14 + Tailwind CSS |
| バックエンド | Node.js + Lambda |
| HTML解析 | cheerio + schema-dts |
| バリデーション | ajv (JSON Schema) |
| キュー | SQS |
| DB | DynamoDB |
| 認証 | NextAuth.js |
| 決済 | Stripe |
| ホスティング | Vercel + AWS |

## リスク

| リスク | 影響 | 対策 |
|-------|------|------|
| Google Rich Results Testとの差別化 | WTP低下 | 「定期監視+アラート」の継続価値 |
| Schema.org仕様の頻繁な更新 | バリデーション陳腐化 | schema-dts自動更新+コミュニティフィードバック |
| SPA/SSRサイトのスキャン困難 | 一部サイト非対応 | Puppeteer fallback（重量級は有料のみ） |
| 構造化データ自体の認知度 | 市場が小さい | SEO教育コンテンツで市場創出 |

## 競合分析

| サービス | 月額 | 弱点 |
|---------|------|------|
| Google Rich Results Test | 無料 | 手動・1ページずつ・監視なし |
| Schema Markup Validator | 無料 | 手動のみ |
| Sitebulb | £13.5〜 | デスクトップツール。自動監視なし |
| ContentKing | $49〜 | 高い。構造化データは機能の一部 |
| Screaming Frog | £199/年 | デスクトップ。スケジュール監視は有料 |

### SchemaLintの優位性
- **$5/月から**: 唯一の低価格構造化データ専門監視SaaS
- **自動定期チェック**: 無料ツールにない「放置しても安心」
- **差分表示**: 何がいつ壊れたか即座にわかる
- **マルチサイト**: Agency向けに1契約で複数サイト

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0（SEO+無料ツール） |
| ARPU | $5〜$12/月 |
| インフラコスト/ユーザー | $0.00〜$0.009/月 |
| 粗利率 | **99.9%** |
| LTV（12ヶ月） | $60〜$144 |
| LTV/CAC | ∞ |
| 月次チャーン | 4%（SEO資産保全は継続必須） |
| $20達成必要人数 | **Pro 4人** |
