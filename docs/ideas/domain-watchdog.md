# 🐕 DomainWatchdog — ドメイン/SSL/DNS一括監視サービス

## 1. 概要・解決する課題

**DomainWatchdog**は、保有ドメインの**期限切れ、SSL証明書の有効期限、DNS変更**を一括監視し、問題発生前にアラートを送るサービス。

### なぜ金を払うか

- ドメイン失効 → **数十万円〜数百万円の取り戻しコスト**（ドロップキャッチ業者に取られる）
- SSL期限切れ → **サイトに「安全ではありません」表示** → 売上直撃
- DNS設定が勝手に変わった → **フィッシングサイトに誘導される可能性**
- **「放っておくと確実に損する」**の典型 = 払う理由が明確
- 月$4で数十万円のリスクを回避

## 2. ターゲットユーザー

### ペルソナA: 複数サービス運営の個人開発者（岡田、34歳）
- ドメイン8個保有（本業+個人プロジェクト）
- Let's Encryptの自動更新が効かなくなって1回痛い目に遭った
- 各レジストラのダッシュボードを見に行くのが面倒

### ペルソナB: Web制作会社の運用担当（小林、29歳）
- クライアントのドメイン20個を管理
- 「ドメイン切れてますよ」とクライアントに怒られたくない
- 複数レジストラ（お名前.com、Cloudflare、AWS Route 53等）に分散

### ペルソナC: スタートアップCTO（吉田、36歳）
- 本番/ステージング/開発で複数ドメイン+サブドメイン
- SSL証明書の有効期限を手動チェックしている
- インシデント防止のために$4なら安い

## 3. 料金プラン

| プラン | 月額 | ドメイン数 | チェック間隔 | 通知先 |
|--------|------|-----------|-------------|--------|
| Free | $0 | 3 | 日次 | Email |
| Pro | $4/月 | 30 | 6時間 | Email/Slack/Webhook |
| Agency | $12/月 | 100 | 1時間 | 上記+PagerDuty+レポート |

年払い: 20%OFF

## 4. ユーザーフロー

```
1. サインアップ（GitHub OAuth）
2. ドメイン追加（ドメイン名を入力するだけ）
3. 自動検出: ドメイン期限、SSL証明書期限、DNS Aレコード/NSレコード
4. 通知先設定（Slack webhook / Email）
5. ダッシュボードで全ドメインのステータス一覧
6. 問題検知時にアラート:
   - 「example.com のSSL証明書が14日後に期限切れです」
   - 「example.com のDNS Aレコードが変更されました: 1.2.3.4 → 5.6.7.8」
   - 「example.com のドメインが30日後に期限切れです」
```

## 5. システムアーキテクチャ

```
[CloudFront + S3: ダッシュボードSPA]
        |
[API Gateway → Lambda: 管理API]
        |
[DynamoDB: ドメイン・ステータスデータ]
        |
[EventBridge Scheduler: 定期チェック]
        |
[Lambda: チェックエンジン]
    |           |           |
[RDAP/WHOIS] [SSL Check] [DNS Lookup]
    |           |           |
    └─────┬─────┘           |
          |                 |
  [DynamoDB: 状態更新]     |
          |                 |
  [変更検知 → 通知Lambda]
          |
  [SES / Slack API / Webhook]
```

## 6. コンポーネント詳細

### 6.1 ドメイン期限チェック
- RDAP API（ICANNの後継プロトコル）でWHOIS情報取得
- `expirationDate`フィールドから期限取得
- 30日前、14日前、7日前、1日前にアラート

### 6.2 SSL証明書チェック
- Node.js `tls.connect()`でSSL証明書を直接取得
- `validTo`フィールドから期限取得
- 30日前、14日前、7日前にアラート
- 証明書チェーンの異常も検知

### 6.3 DNS変更検知
- Node.js `dns.resolve()`でAレコード、NSレコード、MXレコード取得
- 前回チェックとdiff → 変更があれば即通知
- **意図しないDNS変更 = セキュリティインシデントの可能性**

### 6.4 通知システム
- SES: Email通知
- Slack: Incoming Webhook
- Webhook: 汎用（Zapier/n8n等と連携可能）
- 通知テンプレート: ドメイン名、問題種別、残り日数、推奨アクション

### 6.5 ダッシュボード
- ドメイン一覧（ステータス: ✅正常 ⚠️注意 🔴緊急）
- SSL/DNS/ドメイン期限の個別ステータス
- チェック履歴（いつ何がチェックされたか）
- 月次レポート（Agency）

## 7. データベース設計（DynamoDB）

### Users テーブル
```
PK: USER#{userId}
Fields: email, plan, slackWebhookUrl, createdAt
```

### Domains テーブル
```
PK: USER#{userId}
SK: DOMAIN#{domainId}
Fields:
  - domainName: string
  - domainExpiry: string (ISO date)
  - sslExpiry: string (ISO date)
  - sslIssuer: string
  - dnsA: string[] (Aレコード)
  - dnsNS: string[] (NSレコード)
  - dnsMX: string[] (MXレコード)
  - lastCheckedAt: string
  - status: enum (ok|warning|critical)
  - alerts: object ({domain30d: boolean, ssl14d: boolean, ...})
  - createdAt
GSI: status-index (PK: status) — 問題ドメインの一括取得
```

### CheckHistory テーブル
```
PK: DOMAIN#{domainId}
SK: CHECK#{timestamp}
Fields: type (domain|ssl|dns), result, changes
TTL: plan に応じた保持期間（Free 7日, Pro 90日, Agency 1年）
```

## 8. インフラ+AIコスト見積もり

### 想定規模（ローンチ3ヶ月後）
- ユーザー: 40人（Free 30人、Pro 8人、Agency 2人）
- ドメイン数: 250
- チェック/日: 1,000回（ドメイン250 × 3種類 × 頻度考慮）

### AWS コスト
| サービス | 月額見積もり |
|----------|-------------|
| Lambda（チェック+API） | $0.50 |
| DynamoDB（オンデマンド） | $0.50 |
| API Gateway | $0.20 |
| S3 + CloudFront | $0.50 |
| SES | $0.10 |
| EventBridge | $0.10 |
| Route 53 | $0.50 |
| CloudWatch | $0.30 |
| **合計** | **$2.70/月** |

### 外部APIコスト
- RDAP: 無料（ICANN公開API）
- SSL/DNS: Node.js標準ライブラリ（コストゼロ）
- ドメイン: $12/年 = $1/月

### AIコスト
- AI不使用

**総ランニングコスト: 約$3.70/月**

## 9. MVPスコープ

### MVP（2週間）
- GitHub OAuth認証
- ドメインCRUD
- ドメイン期限チェック（RDAP）
- SSL証明書期限チェック
- Email通知
- 最小限ダッシュボード

### v1.1（+1週間）
- DNS変更検知
- Slack通知
- チェック履歴

### v1.2（+1週間）
- Agencyプラン
- 月次レポート
- Webhook通知

## 10. 周知・マーケティング計画

### Phase 1: ローンチ
1. Product Hunt投稿
2. 「ドメイン失効で○○万円損した話」記事（note/Zenn）
3. Twitter/Xで「SSL 期限切れ」「ドメイン 失効」の恐怖訴求
4. Reddit r/webdev、r/sysadmin に投稿

### Phase 2: コンテンツ
1. 「SSL証明書の管理完全ガイド」SEO記事
2. 「Let's Encrypt自動更新が失敗する5つのケース」記事
3. 「ドメイン管理のベストプラクティス」記事

### Phase 3: 継続
1. SEO: 「ドメイン 期限切れ 監視」「SSL 監視」
2. Web制作会社向けDM（Agency プラン）
3. エンジニア向けPodcast/YouTube出演

### 注力チャネル
- **恐怖訴求記事**: 実体験ベースの記事はシェアされやすい
- **SEO**: 「ドメイン 期限切れ」「SSL 期限 確認」は検索ボリュームあり

## 11. 技術スタック

- **Frontend**: React 18 + Vite + TailwindCSS
- **Backend**: Node.js (Lambda)
- **DB**: DynamoDB
- **Auth**: Amazon Cognito + GitHub OAuth
- **Infra**: AWS CDK
- **CI/CD**: GitHub Actions
- **チェック**: Node.js net/tls/dns標準ライブラリ + RDAP

## 12. リスクと対策

| リスク | 影響 | 対策 |
|--------|------|------|
| UptimeRobotがSSL監視を無料で提供 | 差別化消失 | ドメイン期限+DNS変更+SSL の3つを統合した「ドメイン総合監視」で差別化 |
| RDAP APIのレート制限 | チェック失敗 | キャッシュ+分散リクエスト+フォールバック（WHOIS） |
| 誤検知（DNS CDNの変更を「異常」と判定） | 信頼性低下 | CDN（Cloudflare等）のIP帯を除外リストに追加 |
| 「レジストラの通知で十分」と思われる | ユーザーが来ない | 複数レジストラ横断+SSL+DNSの統合管理を訴求 |
| 大量ドメインのチェックでLambda timeout | 機能停止 | SQSで分散処理、1Lambda=1ドメインの並列実行 |

## 13. 競合分析・差別化

| サービス | 価格 | 特徴 | 弱点 |
|----------|------|------|------|
| UptimeRobot | 無料/有料$7~ | HTTP監視メイン、SSL付き | ドメイン期限なし、DNS変更検知なし |
| StatusCake | 無料/有料$20~ | SSL監視あり | ドメイン期限なし、高額 |
| DomainMONSTER | 無料 | ドメイン期限チェック | SSL/DNS なし、通知弱い |
| レジストラの通知 | 無料 | 期限通知あり | 複数レジストラ横断不可、SSL/DNSなし |

### DomainWatchdogの差別化
- **ドメイン期限 + SSL + DNS の3つを一元監視** — これをやっているサービスがほぼない
- **$4/月** — UptimeRobot有料版の半額以下
- **DNS変更検知** — セキュリティ監視としての価値
- **日本語UI** — 日本のWeb制作会社向け
- **複数レジストラ横断** — お名前.com + Cloudflare + Route 53 等を一画面で

## 14. $20/月達成の現実的シナリオ

### シナリオ
- **月1**: ローンチ、恐怖訴求記事 → Free 15人
- **月2**: Product Hunt + SEO → Free 25人、Pro 3人 = **$12/月**
- **月3**: Web制作会社にDM + 口コミ → Free 30人、Pro 5人 = **$20/月** ✅
- **月6**: Free 50人、Pro 10人、Agency 2人 = **$64/月**

### 根拠
- ドメイン保有者は日本だけで数百万人
- 「ドメイン失効」のリスク認知度は高い
- $4/月で「全部まとめて監視」の利便性
- Web制作会社は10-50ドメイン管理が普通 → Agencyプランの需要
- Pro 5人で$20 = 3ヶ月で十分達成可能

**$20/月達成見込み: ローンチ後3ヶ月**
