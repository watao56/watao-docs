# ♿ A11yPing — Webアクセシビリティ自動監視SaaS

## 概要

Webサイトのアクセシビリティ（WCAG 2.1準拠）を定期的に自動スキャンし、違反を検知してアラートを送るSaaS。障害者差別解消法（日本、2024年4月義務化）・ADA（米国）・EAA（EU、2025年6月施行）への対応状況を常時モニタリングし、法的リスクを未然に防ぐ。

## 解決する課題

- **法的リスク**: 日本では2024年4月から合理的配慮の提供が義務化。米国ではADA訴訟が年間4,000件超
- **サイレント違反**: デプロイごとにアクセシビリティが壊れるが、視覚的に気づかない
- **専門知識不足**: WCAG基準は複雑で、非専門家には判断が困難
- **一度きりの監査の限界**: 監査レポートを受けても、次のデプロイで壊れる
- **罰金・訴訟リスク**: EU EAAでは最大€100,000の罰金。米国では平均$25,000の和解金

## ターゲット

| セグメント | ペルソナ | 課題の深刻度 |
|-----------|---------|------------|
| 中小EC事業者 | 法律改正を知らない経営者 | 🔴 罰金・訴訟リスク |
| Web制作会社 | クライアントへの付加価値 | 🔴 納品品質の保証 |
| SaaS企業 | エンタープライズ顧客要件 | 🟡 契約条件充足 |
| 自治体・公共機関 | JIS X 8341-3対応義務 | 🔴 法令遵守 |

## 料金プラン

| プラン | 月額 | ページ数 | スキャン頻度 | レポート |
|--------|------|---------|------------|---------|
| Free | $0 | 5ページ | 月1回 | 基本レポート |
| Pro | $5/月 | 50ページ | 週1回 | 詳細+修正提案 |
| Agency | $15/月 | 500ページ（複数サイト） | 日1回 | PDF出力+クライアント共有 |

### $20達成シナリオ

- **最速**: Pro 4人 = $20/月（2ヶ月目）
- **安定**: Pro 2人 + Agency 1人 = $25/月（3ヶ月目）
- **必要フリーユーザー数**: 40人（有料転換率10%）

## ユーザーフロー

1. **登録**: メールまたはGitHub OAuth
2. **サイト追加**: URLを入力 → 自動クロールでページ一覧取得
3. **初回スキャン**: axe-coreエンジンでWCAG 2.1 AA基準をチェック（30秒〜2分）
4. **結果確認**: 違反一覧（Critical/Major/Minor）+ スクリーンショット + 修正コード例
5. **定期監視**: スケジュールに従い自動スキャン
6. **アラート**: 新規違反検知時にメール/Slack通知
7. **トレンド**: スコア推移グラフで改善状況を可視化

## アーキテクチャ

```
[ユーザーダッシュボード (Vercel)]
        ↓ API
[API Gateway + Lambda]
        ↓
[EventBridge: スキャンスケジューラ]
        ↓
[Lambda: scanner]
    ├── Puppeteer + axe-core (ヘッドレスChrome)
    ├── → DynamoDB (結果保存)
    └── → S3 (スクリーンショット保存)
        ↓ 違反検知時
[Lambda: notifier]
    ├── SES (メール)
    ├── Slack Webhook
    └── Discord Webhook
```

### コンポーネント

| コンポーネント | 技術 | 役割 |
|--------------|------|------|
| フロントエンド | Next.js (Vercel Free) | ダッシュボード |
| スキャナー | Lambda + Puppeteer + axe-core | WCAG違反検出 |
| ストレージ | DynamoDB + S3 | スキャン結果+スクリーンショット |
| スケジューラ | EventBridge | 定期スキャン実行 |
| 通知 | SES + Slack/Discord Webhook | アラート配信 |
| 認証 | NextAuth.js | ユーザー管理 |
| 決済 | Stripe | サブスクリプション |

## DB設計

### DynamoDB テーブル

```
Users
  PK: USER#{userId}
  email, plan, stripeCustomerId, createdAt

Sites
  PK: USER#{userId}
  SK: SITE#{siteId}
  url, name, pageCount, lastScan, score, schedule

Pages
  PK: SITE#{siteId}
  SK: PAGE#{pageUrl}
  lastScan, violationCount, score

ScanResults
  PK: SITE#{siteId}
  SK: SCAN#{timestamp}
  totalViolations, critical, major, minor, score, duration
  TTL: 90日（Free）/ 365日（Pro/Agency）

Violations
  PK: SCAN#{scanId}
  SK: V#{violationId}
  rule, impact, element, selector, fixSuggestion, screenshot
```

### S3バケット

```
a11yping-screenshots/
  {siteId}/{scanId}/{pageUrl-hash}.png
a11yping-reports/
  {siteId}/{scanId}/report.pdf  (Agency)
```

## コスト見積もり

### インフラコスト（月額）

| リソース | 50ユーザー時 | 500ユーザー時 |
|---------|------------|-------------|
| Lambda（スキャン） | $0.00（無料枠内） | $3.00 |
| S3（スクリーンショット） | $0.05 | $0.50 |
| DynamoDB | $0.00（無料枠内） | $2.00 |
| SES | $0.00 | $0.10 |
| Vercel | $0.00 | $0.00 |
| **合計** | **$0.05** | **$5.60** |

### AI使用: なし
axe-core（OSS）がWCAG準拠チェックを行う。ルールベース判定。

## MVPスコープ（2週間）

### Week 1
- [ ] axe-core + Puppeteerでのスキャン実行Lambda
- [ ] ユーザー登録・サイト追加UI
- [ ] スキャン結果表示（違反一覧+スコア）
- [ ] DynamoDBスキーマ

### Week 2
- [ ] 定期スキャン（EventBridge）
- [ ] アラート通知（メール・Slack）
- [ ] スコア推移グラフ
- [ ] Stripe決済・ランディングページ

### MVP後
- PDF レポート出力（Agency向け）
- CI/CD連携（GitHub Actions）
- 修正コード自動生成
- 多言語対応（WCAG基準の日本語解説）

## マーケティング計画

### Phase 1: 法改正訴求（1-4週目）
- 「障害者差別解消法 Webサイト対応チェックリスト」記事
- 「あなたのサイトは大丈夫？無料アクセシビリティ診断」LP
- Web制作系コミュニティでの紹介

### Phase 2: SEO（1-3ヶ月目）
- 「WCAG 2.1 チェックツール」「アクセシビリティ監査」等のキーワード
- 「デプロイでアクセシビリティが壊れる5つのパターン」記事
- 各フレームワーク別（React/Next.js/WordPress）ガイド

### Phase 3: パートナー（3-6ヶ月目）
- Web制作会社への保守メニュー提案
- WordPressプラグインディレクトリ掲載
- 障害者支援団体との連携

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js 14 + Tailwind CSS |
| スキャナー | Puppeteer + axe-core (Lambda Layer) |
| バックエンド | Node.js + Lambda |
| DB | DynamoDB + S3 |
| 認証 | NextAuth.js |
| 決済 | Stripe |
| ホスティング | Vercel + AWS |
| CI/CD | GitHub Actions |

## リスク

| リスク | 影響 | 対策 |
|-------|------|------|
| Puppeteer Lambdaのコールドスタート | スキャン遅延 | Lambda SnapStart、Provisioned Concurrency |
| axe-coreの誤検知 | 信頼性低下 | 手動確認機能、誤検知報告フィードバック |
| 大規模サイトのスキャン時間 | タイムアウト | ページ単位の分散実行、Step Functions |
| 無料ツール（Lighthouse等）との差別化 | WTP低下 | 「定期監視+アラート」の継続価値を訴求 |

## 競合分析

| サービス | 月額 | 弱点 |
|---------|------|------|
| Lighthouse (Google) | 無料 | 手動実行のみ。監視・アラートなし |
| WAVE | 無料 | 1ページずつ手動。定期チェック不可 |
| Siteimprove | $300〜 | エンタープライズ価格。中小には高すぎ |
| Deque axe Monitor | $100〜 | 高価。開発者向けで非技術者には難しい |
| Pa11y | 無料OSS | セルフホスト。管理画面なし |

### A11yPingの優位性
- **$5/月から**: 競合の1/20〜1/60の価格
- **自動定期監視**: 無料ツールにない「放置しても安心」体験
- **法改正対応**: 日本の障害者差別解消法に特化した日本語レポート
- **Web制作会社向けAgency**: 1契約で複数クライアント管理

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0（SEO+コミュニティ） |
| ARPU | $5〜$15/月 |
| インフラコスト/ユーザー | $0.001〜$0.011/月 |
| 粗利率 | **99.9%**（50ユーザー時） |
| LTV（12ヶ月継続） | $60〜$180 |
| LTV/CAC | ∞ |
| 月次チャーン想定 | 3%（法令対応のため解約しづらい） |
| $20達成必要人数 | **Pro 4人** |
