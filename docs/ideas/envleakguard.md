# 🔐 EnvLeakGuard — GitHub秘密情報漏洩検知SaaS

## 1. 概要・解決する課題

**なぜ金を払うか（1行）:** GitHubに誤コミットされたAPIキー・パスワードを即座に検知し、数万ドルの被害を未然に防ぐ。

### 課題の深刻度
- AWS キーの漏洩 → 数時間でクリプトマイニングに悪用、請求額$10,000超の事例多数
- GitGuardianの調査: 2023年にGitHub上で1,000万件以上のシークレットが漏洩
- GitHub Advanced Security（secret scanning for private repos）は **$49/user/月** → 5人チームで$245/月
- 個人開発者・小規模チームには高すぎて使えない

### なぜ「ないと困る」か
1件のキー漏洩で月額の数百〜数千倍の損害が発生する。保険として$5/月は安い。

---

## 2. ターゲットユーザー

### プライマリペルソナ
**田中太郎（32歳）- フリーランスエンジニア**
- 個人で3-5つのプライベートリポジトリを運用
- たまに.envファイルをうっかりコミットしそうになる
- GitHub Advanced Securityは個人には高すぎる
- 過去に一度、AWS keyをpushしてしまい$200の被害

### セカンダリペルソナ
**佐藤花子（28歳）- スタートアップCTO**
- 5人のエンジニアチーム
- GitHub Team plan使用（Advanced Security未契約）
- セキュリティ監査で「秘密情報管理」を指摘された
- 全リポジトリのスキャンが必要だが手動では無理

---

## 3. 料金プラン

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 1リポジトリ、日次スキャン、メール通知 |
| Pro | $5/月 | 10リポジトリ、リアルタイムWebhookスキャン、Slack通知、スキャン履歴30日 |
| Team | $15/月 | 50リポジトリ、チーム管理、監査ログ、優先サポート |

### 価格設定の根拠
- GitHub Advanced Security: $49/user/月 → EnvLeakGuardは1/10以下
- GitGuardian Business: $35/developer/月
- **フリーランス1人が$5/月で全リポジトリ守れる**のは圧倒的に安い

---

## 4. ユーザーフロー

```
1. GitHub OAuth でサインアップ（30秒）
2. 監視したいリポジトリを選択
3. Webhook自動設定 → 以降pushごとにスキャン
4. 漏洩検知時 → Slack/メールで即通知
   - 検知内容: キーの種類、ファイル、コミット、著者
   - 推奨アクション: キーのローテーション手順リンク
5. ダッシュボードでスキャン履歴・統計確認
```

### オンボーディング
- 初回スキャン: 過去100コミットを遡ってスキャン（Freeでも1回だけ）
- 結果に漏洩があれば「今すぐ修正」のガイド付き → 有料変換率UP

---

## 5. システムアーキテクチャ

```
┌─────────┐     ┌──────────────┐     ┌──────────────┐
│  GitHub  │────▶│  Webhook     │────▶│  SQS Queue   │
│  Push    │     │  API (Lambda)│     │              │
└─────────┘     └──────────────┘     └──────┬───────┘
                                            │
                                     ┌──────▼───────┐
                                     │  Scanner     │
                                     │  (Lambda)    │
                                     └──────┬───────┘
                                            │
                      ┌─────────────────────┼─────────────┐
                      │                     │             │
               ┌──────▼───────┐  ┌─────────▼──┐  ┌──────▼──────┐
               │  DynamoDB    │  │  SNS/SES   │  │  Slack      │
               │  (結果保存)   │  │  (通知)     │  │  Webhook    │
               └──────────────┘  └────────────┘  └─────────────┘

┌──────────────┐     ┌──────────────┐
│  Next.js     │────▶│  API Routes  │
│  Frontend    │     │  (認証/CRUD)  │
│  (Vercel)    │     └──────────────┘
└──────────────┘
```

---

## 6. コンポーネント詳細

### 6.1 Webhook受信 (API Gateway + Lambda)
- GitHub Webhookの`push`イベントを受信
- ペイロード検証（X-Hub-Signature-256）
- コミット情報をSQSに投入
- 応答時間: <500ms

### 6.2 シークレットスキャナー (Lambda)
- SQSからコミット情報を取得
- GitHub API でdiffを取得
- 正規表現 + エントロピー分析でシークレット検知
- **検知パターン（150+）:**
  - AWS Access Key/Secret Key
  - GitHub Personal Access Token
  - Slack Bot Token/Webhook URL
  - Stripe API Key
  - Database connection strings
  - Generic high-entropy strings
- 結果をDynamoDBに保存
- 漏洩検知時、SNS/Slackへ通知

### 6.3 フロントエンド (Next.js on Vercel)
- GitHub OAuth認証
- ダッシュボード: リポジトリ一覧、スキャン結果、統計
- 設定: 通知先、除外パターン（false positive対策）
- リポジトリ管理: Webhook有効/無効切り替え

### 6.4 定期スキャン (EventBridge + Lambda)
- Freeプラン用: 日次でリポジトリの最新コミットをスキャン
- 過去スキャンとの差分のみ処理

---

## 7. データベース設計 (DynamoDB)

### Users テーブル
```
PK: USER#{github_user_id}
SK: PROFILE
Attributes:
  - github_username: string
  - email: string
  - plan: "free" | "pro" | "team"
  - slack_webhook_url: string (optional)
  - stripe_customer_id: string
  - created_at: ISO8601
```

### Repositories テーブル
```
PK: USER#{github_user_id}
SK: REPO#{repo_full_name}
Attributes:
  - webhook_id: string
  - is_active: boolean
  - last_scanned_commit: string
  - total_scans: number
  - total_findings: number
```

### ScanResults テーブル
```
PK: REPO#{repo_full_name}
SK: SCAN#{timestamp}#{commit_sha}
Attributes:
  - findings: list[{type, file, line, snippet_masked, severity}]
  - status: "clean" | "alert"
  - scan_duration_ms: number

GSI: UserScans
  PK: USER#{github_user_id}
  SK: SCAN#{timestamp}
```

---

## 8. インフラ+AIコスト見積もり

### 月間想定（有料ユーザー20人、Free 100人時点）

| 項目 | 単価 | 月間使用量 | 月額コスト |
|------|------|-----------|-----------|
| Lambda | $0.20/100万リクエスト | ~50,000リクエスト | $0.01 |
| Lambda コンピュート | $0.0000166667/GB-秒 | 256MB × 2秒 × 50,000 = 25,600GB秒 | $0.43 |
| API Gateway | $3.50/100万リクエスト | ~10,000 | $0.04 |
| DynamoDB | $1.25/WCU, $0.25/RCU | 5 WCU, 10 RCU | $1.56 |
| SQS | $0.40/100万リクエスト | ~100,000 | $0.04 |
| SES | $0.10/1,000通 | ~1,000通 | $0.10 |
| Vercel | Free Hobby plan | - | $0 |
| ドメイン | - | - | ~$1/月（年$12） |
| **合計** | | | **~$2.18/月** |

### AIコスト
- このプロダクトはAIを使わない（正規表現+エントロピー分析）
- AIコスト: **$0**

### GitHub API
- GitHub App: 5,000リクエスト/時間/インストール → 十分

---

## 9. MVPスコープ

### Phase 1: コアMVP（2週間）
- GitHub OAuth + リポジトリ選択UI
- Webhook受信 + SQSキュー
- シークレットスキャナー（上位30パターン）
- メール通知
- シンプルなダッシュボード
- **工数: 10日**

### Phase 2: 有料機能（1週間）
- Stripe決済連携
- Slack通知
- スキャン履歴表示
- 除外パターン設定
- **工数: 5日**

### Phase 3: 成長機能（2週間）
- Team機能（組織管理）
- 過去コミット一括スキャン
- 監査ログ
- **工数: 7日**

---

## 10. 周知・マーケティング計画

### Week 1-2: プレローンチ（開発中）
**Twitter/X:**
- ターゲット: 日本人エンジニア、IndieHackers
- 投稿例1: 「AWSキーをGitHubにpushして$3,000請求された人、僕だけじゃないはず。これを$5/月で防ぐサービス作ってます 🔐 #個人開発」
- 投稿例2: 「GitHub Advanced Securityは$49/user/月。個人開発者には高すぎる。1/10の価格で同じことできるサービスをローンチします」
- 投稿タイミング: 火・木・土の20:00 JST（エンジニアのアクティブ時間帯）

**コミュニティ:**
- **r/selfhosted**, **r/devops** (Reddit): 「I built a secret scanner 10x cheaper than GitHub Advanced Security」
- **Hacker News**: Show HN投稿（英語、グローバル対応後）
- **Zenn**: 「GitHubにシークレットをpushしてしまう事故を$5/月で防ぐ方法」記事
- **Qiita**: 「.envファイル誤コミットの恐怖と対策」記事 → 末尾でサービス紹介

### Week 3-4: ローンチ
- **Product Hunt**: ローンチ投稿（火曜日 0:01 PST）
- **IndieHackers**: 「Revenue: $0 → $20 in 30 days」チャレンジ投稿
- **Dev.to**: 英語記事「Why I Built a $5/mo GitHub Secret Scanner」

### 継続施策
- 月1回のZenn/Qiita記事
- Twitter: 週3回のシークレット漏洩ニュース共有 + 啓蒙
- GitHub Actionsとしても無料版提供 → 有料版へのファネル

---

## 11. 技術スタック

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 14 (App Router) |
| Hosting | Vercel (Hobby) |
| Auth | NextAuth.js + GitHub OAuth |
| Backend | AWS Lambda (Node.js 20) |
| Queue | Amazon SQS |
| Database | Amazon DynamoDB |
| 通知 | Amazon SES + Slack Webhook |
| 決済 | Stripe |
| IaC | AWS CDK (TypeScript) |
| CI/CD | GitHub Actions |

---

## 12. リスクと対策

| リスク | 深刻度 | 対策 |
|--------|--------|------|
| GitHub APIレート制限 | High | GitHub App使用（5,000req/h/install）、差分のみスキャン |
| False positive多すぎ | High | エントロピー閾値チューニング、ユーザー除外パターン、フィードバックループ |
| GitHub自体がsecret scanning強化 | Medium | 差別化: 価格（Free枠でprivate対応）、Slack通知、カスタムパターン |
| セキュリティ（自分がコード閲覧権限持つ） | High | diffのみ取得、コード本体は保存しない、SOC2準拠を目指す旨を明記 |
| Stripe決済の日本対応 | Low | Stripe Japanは問題なく使える |
| スケール時のLambda同時実行数 | Low | SQSバッファリングで制御、reserved concurrency設定 |

---

## 13. 競合分析・差別化

| 機能 | EnvLeakGuard | GitHub Advanced Security | GitGuardian | TruffleHog (OSS) |
|------|-------------|-------------------------|-------------|-------------------|
| Private repo スキャン | ✅ Free枠あり | $49/user/月 | 個人Free、Business $35/dev/月 | ✅ CLI手動 |
| リアルタイムWebhook | ✅ | ✅ | ✅ | ❌ |
| Slack通知 | ✅ Pro | ✅ | ✅ Business | ❌ |
| カスタムパターン | ✅ Pro | ✅ | ✅ Business | ✅ |
| セットアップ | 30秒 | GitHub Enterprise必要 | 5分 | CLI install + 設定 |
| 最低月額（個人） | **$0-$5** | **$49** | **$0（制限あり）** | **$0** |

### なぜ勝てるか
1. **価格**: GitHub Advanced Securityの1/10、GitGuardianの個人Free枠はpublic repoのみ
2. **シンプルさ**: 30秒でセットアップ完了、余計な機能なし
3. **ニッチ特化**: 個人開発者・小規模チームだけに集中（大企業は狙わない）
4. **GitGuardian Free との差**: GitGuardianはFreeだと3人まで・internal monitoringのみ。EnvLeakGuardは10リポジトリまでPro機能

---

## 14. $20/月達成の現実的シナリオ

### 前提
- Free → Pro変換率: 10%
- 月次解約率: 5%

### 月次モデル
| 月 | 新規Free | 累計Free | Pro変換 | 累計Pro | MRR |
|----|---------|---------|---------|---------|-----|
| 1 | 50 | 50 | 5 | 5 | $25 ✅ |
| 2 | 40 | 85 | 4 | 9 | $45 |
| 3 | 60 | 135 | 6 | 14 | $70 |

### 新規Free 50人/月の根拠
- Zenn記事: 1記事で1,000PV → CTR 5% → 50人（過去実績ベース）
- Twitter: フォロワー連鎖で20-30人
- Reddit/HN: 1投稿で30-50人
- 初月は記事+SNS同時展開で50人は保守的な見積もり

### $20達成: **1ヶ月目**
Proユーザー4人で$20達成。Free50人の10%変換=5人。

---

## 15. ユニットエコノミクス

### Proユーザー1人あたり（月額）
| 項目 | コスト |
|------|--------|
| Lambda (スキャン) | $0.02 |
| DynamoDB | $0.05 |
| SES (通知) | $0.01 |
| API Gateway | $0.01 |
| **合計コスト** | **$0.09** |
| **収益** | **$5.00** |
| **粗利** | **$4.91 (98.2%)** |

### Freeユーザー1人あたり（月額）
| 項目 | コスト |
|------|--------|
| Lambda (日次スキャン) | $0.005 |
| DynamoDB | $0.01 |
| **合計コスト** | **$0.015** |

### LTV（Life Time Value）
- 平均継続: 12ヶ月（月次解約率5%想定）
- LTV = $5 × 12 = $60
- CAC（獲得コスト）= $0（オーガニックのみ）
- **LTV/CAC = ∞**（有料マーケティング不使用時）

### 損益分岐点
- 固定費: ドメイン$1/月
- 変動費: 1人あたり$0.09/月
- **1人のProユーザーで黒字**
