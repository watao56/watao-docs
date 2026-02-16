# ⏰ CronFailsafe — cronジョブ死活監視サービス

## 1. 概要・解決する課題

**CronFailsafe**は、cronジョブや定期バッチ処理が「実行されなかった」ことを即座に検知・通知するSaaS。

### なぜ金を払うか

- cronが止まっていることに**数日〜数週間気づかない**のは現実によくある
- バックアップが取れていない、レポートが送られていない、データ同期が止まっている——気づいた時には手遅れ
- **「動かなかったら損する」**処理を確実に監視するためのサービス
- 月$3-5で数万円の損害を防げる = ROI明確

### 仕組み

ジョブの最後に `curl https://cronfailsafe.com/ping/{token}` を追加するだけ。期待時間内にpingが来なければSlack/Email/Webhookで通知。

## 2. ターゲットユーザー

### ペルソナA: 個人開発者・フリーランスエンジニア（田中、32歳）
- VPS 2-3台で自作サービスを運用
- cronで日次バックアップ、API同期、レポート送信
- 以前cronが2週間止まっていてDBバックアップが取れていなかった苦い経験あり
- Healthchecks.ioは知っているがセルフホストは面倒、SaaSは$17/月と高い

### ペルソナB: スタートアップのインフラ担当（鈴木、28歳）
- 社内の定期バッチ処理を15本程度管理
- DatadogやPagerDutyは導入しているが、cron特化の監視がほしい
- 月$5で済むなら稟議不要で使える

## 3. 料金プラン

| プラン | 月額 | ジョブ数 | 通知先 | 保持期間 |
|--------|------|----------|--------|----------|
| Free | $0 | 20 | Email 1件 | 7日 |
| Pro | $4/月 | 100 | Slack/Email/Webhook 無制限 + 週次レポート | 90日 |
| Team | $12/月 | 500 | 上記+PagerDuty + チームメンバー管理 | 1年 |

年払い割引: 20%OFF

## 4. ユーザーフロー

```
1. サインアップ（GitHub OAuth）
2. ジョブ作成（名前、期待間隔、猶予時間を設定）
3. 表示されるURLをcronの最後に追加
4. 通知先を設定（Slack webhook / Email）
5. ダッシュボードでジョブのステータス確認
6. ジョブが期待時間内にpingしなかった → 即座に通知
```

## 5. システムアーキテクチャ

```
[cron/バッチ] --HTTP GET/POST--> [API Gateway]
                                      |
                                 [Lambda: ping受信]
                                      |
                                 [DynamoDB: ジョブ状態]
                                      |
                          [EventBridge Scheduler: 定期チェック]
                                      |
                            [Lambda: 遅延検知+通知]
                                      |
                          [SNS / SES / Slack API]

[CloudFront + S3: SPA ダッシュボード]
     |
[API Gateway → Lambda: CRUD API]
```

## 6. コンポーネント詳細

### 6.1 Ping受信API
- `GET/POST /ping/{token}` — ジョブのping受信
- DynamoDBの`lastPingAt`を更新
- レスポンス: 200 OK（最小レイテンシ）

### 6.2 遅延検知エンジン
- EventBridge Schedulerで1分ごとに実行
- 各ジョブの`lastPingAt + interval + grace` < now を判定
- 条件合致 → 通知キューにプッシュ

### 6.3 通知システム
- SES: Email通知
- Slack Incoming Webhook: Slack通知
- SNS: Webhook通知
- 通知の重複防止（同一アラートは再通知間隔を設定）

### 6.4 ダッシュボード（SPA）
- React + Vite
- ジョブ一覧、ステータス、履歴表示
- ジョブ作成/編集/削除

### 6.5 認証
- GitHub OAuth（Cognito利用）
- JWTトークン

## 7. データベース設計（DynamoDB）

### Users テーブル
```
PK: USER#{userId}
Fields: email, plan, slackWebhookUrl, createdAt
```

### Jobs テーブル
```
PK: USER#{userId}
SK: JOB#{jobId}
Fields: name, token (unique), intervalSeconds, graceSeconds, lastPingAt, status (ok|late|new), notifyChannels[], createdAt
GSI: token-index (PK: token) — ping受信時のルックアップ用
```

### PingHistory テーブル
```
PK: JOB#{jobId}
SK: PING#{timestamp}
Fields: source_ip, duration_ms
TTL: plan に応じた保持期間
```

## 8. インフラ+AIコスト見積もり

### 想定規模（ローンチ3ヶ月後）
- ユーザー: 50人（Free 40人、Pro 8人、Team 2人）
- ジョブ数: 200
- Ping/日: 2,000回

### AWS コスト
| サービス | 月額見積もり |
|----------|-------------|
| Lambda（ping + チェック + API） | $0.50 |
| DynamoDB（オンデマンド） | $1.00 |
| API Gateway | $0.30 |
| S3 + CloudFront | $0.50 |
| SES（Email通知） | $0.10 |
| EventBridge Scheduler | $0.10 |
| Cognito | $0.00（50,000 MAUまで無料） |
| Route 53 | $0.50 |
| CloudWatch Logs | $0.50 |
| **合計** | **$3.50/月** |

### AIコスト
- このプロダクトにAI機能は不要
- 開発時のAIコスト: Claude Code利用 $20-30（一回限り）

### ドメイン
- cronfailsafe.com: $12/年 = $1/月

**総ランニングコスト: 約$4.50/月**

## 9. MVPスコープ

### MVP（2週間）
- [x] GitHub OAuth認証
- [x] ジョブCRUD API
- [x] Ping受信エンドポイント
- [x] 1分間隔の遅延チェック
- [x] Email通知
- [x] 最小限のダッシュボード（ジョブ一覧+ステータス）

### v1.1（+1週間）
- [ ] Slack通知
- [ ] Webhook通知
- [ ] Ping履歴グラフ

### v1.2（+1週間）
- [ ] Teamプラン
- [ ] PagerDuty連携

## 10. 周知・マーケティング計画

### Phase 1: ローンチ前（1週間）
1. ランディングページ作成（cronfailsafe.com）
2. 個人ブログに「cronが止まっていた恐怖体験」記事投稿
3. Twitter/Xで開発過程を発信

### Phase 2: ローンチ（1-2週間）
1. **Product Hunt**に投稿
2. **Hacker News** Show HNに投稿
3. Reddit r/selfhosted、r/devops に投稿
4. Dev.to / Zenn に技術記事投稿「cronジョブ監視のベストプラクティス」
5. 日本のエンジニアコミュニティ（Qiita、Zenn）に記事

### Phase 3: 継続（月次）
1. 「cron monitoring」「cron job alert」でSEO記事
2. GitHub Actions / GitLab CI連携の記事
3. 満足ユーザーにTwitterシェア依頼

### 想定獲得チャネル
- SEO: 50%
- Product Hunt / HN: 30%
- 口コミ: 20%

## 11. 技術スタック

- **Frontend**: React 18 + Vite + TailwindCSS
- **Backend**: Node.js (Lambda)
- **DB**: DynamoDB
- **Auth**: Amazon Cognito + GitHub OAuth
- **Infra**: AWS CDK (TypeScript)
- **CI/CD**: GitHub Actions
- **Monitoring**: CloudWatch
- **Domain**: Route 53

## 12. リスクと対策

| リスク | 影響 | 対策 |
|--------|------|------|
| Healthchecks.ioの無料プランで十分 | ユーザーが有料にしない | Free枠を3ジョブに絞り、Pro$4の安さで差別化 |
| Lambda cold startでping応答が遅い | ユーザー体験低下 | Provisioned Concurrency（コスト増）or SnapStart |
| DynamoDBコスト増大 | 赤字 | TTLで古いデータ自動削除、オンデマンド課金 |
| 通知が届かない（致命的） | 信頼性失墜 | 通知失敗時のリトライ機構、通知自体の監視 |
| 競合の価格引き下げ | 価格競争 | 機能ではなくシンプルさで差別化 |

## 13. 競合分析・差別化

| サービス | 価格 | 特徴 | 弱点 |
|----------|------|------|------|
| Healthchecks.io | 無料(20件)/有料$17~ | OSS、セルフホスト可 | SaaS版は$17〜と高い |
| Cronitor | $10/月〜 | 高機能 | 個人には高い |
| Dead Man's Snitch | $5/月〜 | シンプル | ジョブ5件で$5 |
| UptimeRobot | 無料あり | HTTP監視メイン | cron特化ではない |

### CronFailsafeの差別化
- **Free 20ジョブ** — Healthchecks.ioと同等の無料枠で導入ハードル最小化
- **$4/月で100ジョブ+週次レポート** — Pro枠の圧倒的コスパ
- **Slack/Email/Webhook** — 必要十分な通知先
- **セットアップ30秒** — curl一行追加するだけ
- **日本語UI・日本語サポート** — 日本市場でニッチ獲得（競合は全て英語）
- **cron以外も対応** — GitHub Actions、Cloud Functions、AWS Lambda等の定期実行もカバー
- **週次サマリーレポート** — Slack/Emailで1週間の実行状況を自動レポート（Pro以上）

## 14. $20/月達成の現実的シナリオ

### シナリオ
- **月1**: ローンチ、Product Hunt/HN投稿 → Free 20人獲得
- **月2**: SEO記事公開、Zenn記事 → Free 40人、Pro 3人 = **$12/月**
- **月3**: 口コミ、追加記事 → Free 60人、Pro 5人 = **$20/月** ✅
- **月6**: Free 100人、Pro 10人、Team 1人 = **$52/月**

### 根拠
- Healthchecks.ioのGitHub Stars 8k+ = 需要は確実に存在
- $4/月はほぼ衝動買いレベル
- cronを使うエンジニアは日本だけで数十万人
- 3ジョブFreeで試して便利→4つ目のジョブ追加時にPro化する導線

**$20/月達成見込み: ローンチ後2-3ヶ月**
