# 🤖 RoboGuard — robots.txt / sitemap.xml 破損検知SaaS

## 概要

Webサイトのrobots.txt と sitemap.xml を定期監視し、意図しない変更・破損・消失を検知してアラートを送るSaaS。「デプロイでrobots.txtにDisallow: /が入ってGoogleインデックスが全消失」「sitemap.xmlが壊れてクロール効率が激減」といったSEO事故を未然に防ぐ。

## 解決する課題

- **robots.txt誤設定でインデックス消失**: `Disallow: /` が紛れ込むとサイト全体がGoogle検索から消える。復旧に数週間〜数ヶ月
- **sitemap.xml破損**: XMLパースエラー、404、URLの欠落でクロール効率が激減
- **デプロイ時の上書き事故**: CI/CDパイプラインやCMS更新でファイルが消える/上書きされる
- **気づくのが遅い**: Search Consoleで気づく頃には検索流入が激減した後
- **被害額**: SEO流入が月100万円のサイトで1週間のインデックス消失 = 約25万円の損失

## ターゲット

| セグメント | ペルソナ | 課題の深刻度 |
|-----------|---------|------------|
| SEO担当者 | 検索流入が売上に直結する事業 | 🔴 流入消失=売上直撃 |
| Web制作会社 | クライアントサイトの保守管理 | 🔴 事故=信頼喪失+賠償 |
| メディア/ブログ運営者 | 広告収入がSEO依存 | 🔴 インデックス消失=収入ゼロ |
| ECサイト運営者 | 商品ページの検索露出 | 🟡 商品が検索に出ない |

## 料金プラン

| プラン | 月額 | サイト数 | 監視間隔 | 機能 |
|--------|------|---------|---------|------|
| Free | $0 | 1 | 日1回 | 基本チェック+メール通知 |
| Pro | $4/月 | 10 | 1時間 | 差分表示+Slack+履歴30日 |
| Agency | $12/月 | 50 | 15分 | 全機能+クライアント別レポート |

### $20達成シナリオ

- **最速**: Pro 5人 = $20/月（2ヶ月目）
- **安定**: Pro 3人 + Agency 1人 = $24/月（3ヶ月目）
- **必要フリーユーザー数**: 40人（有料転換率10-12%）

## ユーザーフロー

1. **登録**: メール or Google OAuth
2. **サイト追加**: ドメインURL入力 → 自動でrobots.txt/sitemap.xml検出
3. **ベースライン取得**: 現在の正常状態をスナップショット保存
4. **定期監視**: スケジュールに従いファイルを取得・比較
5. **変更検知**: diff表示（追加行=緑、削除行=赤）
6. **危険度判定**: Critical（Disallow: / 追加）、Warning（UA変更）、Info（コメント変更）
7. **アラート**: 危険度に応じてメール/Slack/Discord通知
8. **履歴**: 過去の変更をタイムライン表示、任意時点にロールバック参照

## アーキテクチャ

```
[EventBridge: スケジューラ]
        ↓
[Lambda: fetcher]
    ├── robots.txt 取得 + パース
    ├── sitemap.xml 取得 + XMLバリデーション
    └── 前回スナップショットと差分比較
        ↓ 変更検知時
[Lambda: analyzer]
    ├── 危険度判定（ルールベース）
    ├── → DynamoDB (結果保存)
    └── → S3 (スナップショット保存)
        ↓ Critical/Warning時
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
  domain, robotsUrl, sitemapUrl, lastCheck, status

Snapshots
  PK: SITE#{siteId}
  SK: SNAP#{timestamp}
  robotsHash, sitemapHash, robotsS3Key, sitemapS3Key

Changes
  PK: SITE#{siteId}
  SK: CHG#{timestamp}
  fileType, severity, diff, description
  TTL: 30日(Free) / 365日(Pro/Agency)
```

### S3
```
roboguard-snapshots/
  {siteId}/{timestamp}/robots.txt
  {siteId}/{timestamp}/sitemap.xml
```

## コスト見積もり

| リソース | 50ユーザー時 | 500ユーザー時 |
|---------|------------|-------------|
| Lambda | $0.00 | $0.50 |
| DynamoDB | $0.00 | $1.50 |
| S3 | $0.01 | $0.10 |
| SES | $0.00 | $0.05 |
| Vercel | $0.00 | $0.00 |
| **合計** | **$0.01** | **$2.15** |

### AI使用: なし
テキスト差分比較 + XMLバリデーション + ルールベース危険度判定のみ。

## MVPスコープ（2週間）

### Week 1
- [ ] robots.txt/sitemap.xml取得・パースLambda
- [ ] 差分検知ロジック（テキストdiff + 危険ルール判定）
- [ ] ユーザー登録・サイト追加UI
- [ ] DynamoDB + S3 スナップショット保存

### Week 2
- [ ] 定期監視（EventBridge）
- [ ] アラート通知（メール・Slack）
- [ ] 変更履歴タイムライン表示
- [ ] Stripe決済・LP

### MVP後
- sitemap.xml内のURL数推移グラフ
- Search Console API連携（インデックス状況）
- CI/CD連携（デプロイ前チェック）
- WordPress プラグイン

## マーケティング計画

### Phase 1: 恐怖訴求（1-4週目）
- 「robots.txtの1行でSEO流入がゼロになった実話」ブログ記事
- SEO系コミュニティ（SEO Japan、海外SEOフォーラム）での紹介
- Twitter/Xで「robots.txt事故」事例のスレッド

### Phase 2: SEO（1-3ヶ月目）
- 「robots.txt チェッカー」「sitemap.xml バリデーター」でSEO
- 無料ツールとして1回限りチェック機能を公開（リード獲得）
- 「デプロイでSEOを壊さないためのチェックリスト」記事

### Phase 3: 制作会社チャネル（3-6ヶ月目）
- Web制作会社の保守メニューに「SEOファイル監視」を提案
- Agency向けホワイトラベル機能

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js 14 + Tailwind CSS |
| バックエンド | Node.js + Lambda |
| DB | DynamoDB + S3 |
| 差分検知 | diff (npm) + fast-xml-parser |
| 認証 | NextAuth.js |
| 決済 | Stripe |
| ホスティング | Vercel + AWS |

## リスク

| リスク | 影響 | 対策 |
|-------|------|------|
| PageDiffとの機能重複感 | 差別化必要 | SEO特化（robots.txt/sitemap専門）を明確化 |
| robots.txt未設置サイト | 監視対象なし | sitemap.xmlのみ監視モード提供 |
| 大規模sitemap（10万URL） | パース時間 | sitemap index対応、サンプリング検証 |
| CDNキャッシュで変更検知遅延 | アラート遅れ | Cache-Control: no-cache ヘッダ付きリクエスト |

## 競合分析

| サービス | 月額 | 弱点 |
|---------|------|------|
| Google Search Console | 無料 | 事後レポート。リアルタイム検知なし |
| Screaming Frog | £199/年 | デスクトップツール。自動監視なし |
| SEMrush Site Audit | $130〜 | 高い。robots.txt監視は機能の一部 |
| Ahrefs | $99〜 | 高い。同上 |
| ContentKing | $49〜 | robots.txt監視あるが高価 |

### RoboGuardの優位性
- **$4/月から**: SEOツールの1/10〜1/30の価格
- **robots.txt/sitemap特化**: 余計な機能なし。設定10秒
- **即座にアラート**: Search Consoleでは数日遅れる事象を即検知
- **差分表示**: 何が変わったか一目瞭然

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0（SEO+コミュニティ） |
| ARPU | $4〜$12/月 |
| インフラコスト/ユーザー | $0.0002〜$0.004/月 |
| 粗利率 | **99.9%** |
| LTV（12ヶ月） | $48〜$144 |
| LTV/CAC | ∞ |
| 月次チャーン | 4%（SEO依存事業は解約しづらい） |
| $20達成必要人数 | **Pro 5人** |
