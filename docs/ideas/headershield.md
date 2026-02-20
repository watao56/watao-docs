# 🛡️ HeaderShield — HTTPセキュリティヘッダー監視SaaS

## 概要

WebサイトのHTTPセキュリティヘッダー（CSP, HSTS, X-Frame-Options, Permissions-Policy等）を定期監視し、設定漏れ・変更・劣化を検知してアラートを送るSaaS。「CDN設定変更でHSTSが消えた」「デプロイでCSPが壊れた」「セキュリティスコアがF→Aに戻せない」といった問題を自動検知。

## 解決する課題

- **セキュリティヘッダー設定漏れ**: 新規サイトの70%以上がCSP未設定（SecurityHeaders.com調査）
- **デプロイ/CDN変更での意図しない消失**: Cloudflare設定変更、nginx更新でヘッダーが消えるケースが頻発
- **XSS/クリックジャッキング脆弱性**: CSP/X-Frame-Options未設定=攻撃対象
- **ブラウザ警告**: HSTS未設定だとChrome警告表示でユーザー離脱
- **監査・コンプライアンス不適合**: PCI DSS、SOC2等でセキュリティヘッダーは必須項目
- **SecurityHeaders.comは手動のみ**: 毎日チェックする人はいない

## ターゲット

| セグメント | ペルソナ | 課題の深刻度 |
|-----------|---------|------------|
| Web制作会社 | クライアント保守担当 | 🔴 脆弱性=賠償リスク |
| SaaS企業 | セキュリティ担当 | 🔴 SOC2/PCI DSS準拠 |
| ECサイト運営者 | 顧客情報を扱う事業 | 🔴 情報漏洩=信頼喪失 |
| フリーランス開発者 | 複数サイト管理 | 🟡 品質保証 |

## 料金プラン

| プラン | 月額 | サイト数 | 監視間隔 | 機能 |
|--------|------|---------|---------|------|
| Free | $0 | 1 | 週1回 | 基本スコア+メール |
| Pro | $5/月 | 20 | 日1回 | 全ヘッダー+Slack+履歴 |
| Agency | $15/月 | 100 | 6時間 | PDF報告書+クライアント管理 |

### $20達成シナリオ

- **最速**: Pro 4人 = $20/月（2ヶ月目）
- **安定**: Pro 2人 + Agency 1人 = $25/月（3ヶ月目）
- **必要フリーユーザー数**: 50人（有料転換率8%）

## ユーザーフロー

1. **登録**: メール or GitHub OAuth
2. **サイト追加**: URLを入力 → 即座に初回スキャン実行
3. **スコア表示**: A+〜F のセキュリティスコア（SecurityHeaders.com互換）
4. **ヘッダー一覧**: 各ヘッダーの設定状況（✅設定済み / ⚠️不十分 / ❌未設定）
5. **修正ガイド**: ヘッダーごとの推奨設定+コピペ可能な設定例（nginx/Apache/Cloudflare）
6. **定期監視**: スケジュールに従い自動チェック
7. **アラート**: スコア低下 or ヘッダー消失時に通知

## アーキテクチャ

```
[EventBridge: スケジューラ]
        ↓
[Lambda: header-scanner]
    ├── HTTP HEAD/GETリクエスト
    ├── レスポンスヘッダー解析
    ├── スコア算出（ルールベース）
    └── 前回結果と比較
        ↓ 変更/劣化検知時
    ┌───┴───┐
[DynamoDB]  [Lambda: notifier]
             ├── SES
             ├── Slack
             └── Discord

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
  url, lastScan, currentScore, previousScore

ScanResults
  PK: SITE#{siteId}
  SK: SCAN#{timestamp}
  score, headers: {
    csp: { present, value, rating },
    hsts: { present, value, rating },
    xFrameOptions: { present, value, rating },
    xContentTypeOptions: { present, value, rating },
    permissionsPolicy: { present, value, rating },
    referrerPolicy: { present, value, rating },
    crossOriginPolicies: { present, value, rating }
  }
  TTL: 30日(Free) / 365日(Pro/Agency)

Alerts
  PK: USER#{userId}
  SK: ALERT#{timestamp}
  siteId, type, oldScore, newScore, changedHeaders
```

## コスト見積もり

| リソース | 50ユーザー時 | 500ユーザー時 |
|---------|------------|-------------|
| Lambda | $0.00 | $0.20 |
| DynamoDB | $0.00 | $1.00 |
| SES | $0.00 | $0.05 |
| Vercel | $0.00 | $0.00 |
| **合計** | **$0.00** | **$1.25** |

### AI使用: なし
HTTPヘッダー解析 + ルールベーススコアリングのみ。外部API不使用。

## MVPスコープ（10日）

### Week 1
- [ ] HTTPヘッダー取得・解析Lambda
- [ ] スコアリングロジック（7ヘッダー×5段階）
- [ ] ユーザー登録・サイト追加UI
- [ ] 結果表示ダッシュボード

### Week 2（前半）
- [ ] 定期監視（EventBridge）
- [ ] アラート通知
- [ ] 修正ガイド表示（nginx/Apache/Cloudflare別）
- [ ] Stripe決済・LP

### MVP後
- スコア推移グラフ
- PDFレポート出力
- CI/CD連携（GitHub Actions）
- CSP自動生成ツール

## マーケティング計画

### Phase 1: 無料ツール（1-2週目）
- SecurityHeaders.comライクな1回限り無料チェックツール公開
- 「あなたのサイトのセキュリティスコアは？」バイラルLP
- Product Hunt・IndieHackers投稿

### Phase 2: SEO（1-3ヶ月目）
- 「CSP 設定方法」「HSTS 設定」「セキュリティヘッダー nginx」記事群
- 「Webサイトセキュリティヘッダー完全ガイド」ロングコンテンツ
- 各ホスティング別（Vercel/Netlify/Cloudflare）設定記事

### Phase 3: B2B（3-6ヶ月目）
- Web制作会社の保守オプション提案
- セキュリティ監査会社との連携
- PCI DSS準拠チェックリスト連携

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js 14 + Tailwind CSS |
| バックエンド | Node.js + Lambda |
| DB | DynamoDB |
| 認証 | NextAuth.js |
| 決済 | Stripe |
| ホスティング | Vercel + AWS |

## リスク

| リスク | 影響 | 対策 |
|-------|------|------|
| SecurityHeaders.comが監視機能追加 | 競合激化 | 価格優位性+修正ガイドで差別化 |
| CDN/WAFがヘッダーを付与するケース | 誤判定 | 実際のレスポンスヘッダーを正確に取得 |
| 「セキュリティヘッダーなんて知らない」層 | 市場認知不足 | 教育コンテンツで啓蒙+無料ツールで接点 |
| ブラウザの仕様変更 | ルール更新必要 | MDN/Chrome DevRelのフォロー体制 |

## 競合分析

| サービス | 月額 | 弱点 |
|---------|------|------|
| SecurityHeaders.com | 無料 | 手動のみ。監視・アラートなし |
| Mozilla Observatory | 無料 | 手動のみ。更新停滞気味 |
| Detectify | $85〜 | 高い。ヘッダー監視は機能の一部 |
| Qualys SSL Labs | 無料 | SSL特化。他ヘッダーは対象外 |
| UpGuard | $5,000〜/年 | エンタープライズ向け |

### HeaderShieldの優位性
- **$5/月から**: 「SecurityHeaders.com + 自動監視」の唯一の低価格SaaS
- **修正ガイド付き**: 問題検知だけでなく「どう直すか」まで提示
- **マルチサイト対応**: 制作会社が20サイトを一括管理
- **スコア推移**: 改善の進捗を可視化（報告書にも使える）

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0（SEO+無料ツール） |
| ARPU | $5〜$15/月 |
| インフラコスト/ユーザー | $0.00〜$0.0025/月 |
| 粗利率 | **99.9%** |
| LTV（12ヶ月） | $60〜$180 |
| LTV/CAC | ∞ |
| 月次チャーン | 4%（セキュリティは継続必須） |
| $20達成必要人数 | **Pro 4人** |
