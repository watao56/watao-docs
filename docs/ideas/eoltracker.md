# ⏳ EOLTracker — 技術スタックEOL監視

## 概要

プロジェクトで使用しているプログラミング言語、フレームワーク、OS、データベースのEnd of Life（サポート終了）日を自動追跡し、EOL前にアラートを送るサービス。EOLを過ぎたソフトウェアはセキュリティパッチが提供されず、**脆弱性が放置=個人情報漏洩=損害賠償**のリスク。endoflife.date API（無料・オープンソース）を活用し、300+の製品のEOL情報を自動取得。

## ターゲット

- **メイン**: 1-20人規模の開発チーム/Web制作会社（技術的負債の管理ができていない）
- **サブ**: フリーランスエンジニア（複数クライアントのプロジェクトを管理）
- **拡張**: SIer・受託開発会社（納品後の保守計画に利用）

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 2プロジェクト、メール通知、月次チェック |
| Pro | $5/月 | 10プロジェクト、Slack/メール通知、週次チェック、タイムライン表示 |
| Team | $12/月 | 無制限プロジェクト、チーム共有、GitHub連携(package.json/Gemfile/requirements.txt自動読取)、CSV出力 |

## ユーザーフロー

1. **登録**: メール/GitHubで登録
2. **プロジェクト作成**: プロジェクト名を入力
3. **技術スタック追加**: 使用技術＋バージョンを選択（オートコンプリート、300+対応）
   - 例: Node.js 18, Ubuntu 22.04, PostgreSQL 15, React 18
4. **または自動検出**: GitHubリポジトリURLを入力→package.json等からバージョンを自動抽出
5. **監視開始**: endoflife.date APIで各技術のEOL日を自動取得
6. **アラート**: EOL 6ヶ月前、3ヶ月前、1ヶ月前に通知
7. **ダッシュボード**: 全プロジェクトの技術スタックEOLタイムライン

## アーキテクチャ

```
[endoflife.date API] ← 無料、300+製品
         ↓
[Lambda: EOL Data Syncer]
(EventBridge: 週1回)
         ↓
[DynamoDB: EOLデータキャッシュ]
         ↓
[Lambda: Alert Checker]
(EventBridge: 毎日)
         ↓
判定: EOL日 - 今日 < 閾値?
         ↓ Yes
[SES / Slack Webhook]

[GitHub API] → [Lambda: Stack Detector]
                (package.json, Gemfile等を解析)
                → [DynamoDB: プロジェクト技術スタック]
```

### コンポーネント

- **Frontend**: Next.js（Vercel無料枠）
- **API**: AWS Lambda + API Gateway
- **DB**: DynamoDB
- **EOLデータ**: endoflife.date API（無料、レート制限なし）
- **GitHub連携**: GitHub REST API（public repos: 認証不要）
- **スケジューラ**: EventBridge
- **通知**: SES + Slack Webhook
- **認証**: NextAuth.js（メール + GitHub OAuth）

## DB設計

### Users テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | UUID |
| email | String | メール |
| plan | String | free/pro/team |
| slackWebhook | String | Slack通知先 |

### Projects テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | ユーザーID |
| projectId (SK) | String | プロジェクトID |
| name | String | プロジェクト名 |
| repoUrl | String | GitHubリポジトリURL（任意） |
| lastSynced | Number | 最終同期日時 |

### ProjectStacks テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| projectId (PK) | String | プロジェクトID |
| stackId (SK) | String | スタックID |
| product | String | 製品名（nodejs, ubuntu等） |
| version | String | バージョン（18, 22.04等） |
| eolDate | String | EOL日 |
| ltsUntil | String | LTS期限（あれば） |
| status | String | supported/warning/eol |

### EOLCache テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| product (PK) | String | 製品名 |
| version (SK) | String | バージョン |
| eolDate | String | EOL日 |
| ltsUntil | String | LTS期限 |
| updatedAt | Number | キャッシュ更新日 |

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
- endoflife.date API: **$0**（無料・オープンソース）
- GitHub API: **$0**（public repos、認証不要）

### AIコスト
- AI不使用。

### 200ユーザー時
| 項目 | 月額 |
|------|------|
| Lambda | $0.30 |
| DynamoDB | $0.40 |
| API Gateway | $0.80 |
| **合計** | **$1.50/月** |

## MVPスコープ

### Phase 1（10日）
- ユーザー登録/ログイン
- プロジェクト作成
- 技術スタック手動追加（オートコンプリート、endoflife.date APIからの製品一覧）
- EOL日自動取得＋キャッシュ
- アラート通知（メール）
- ダッシュボード（プロジェクト一覧、EOLステータスカラー表示）

### Phase 2（+5日）
- GitHub連携（package.json自動読取）
- Slack通知
- EOLタイムライン表示（ガントチャート風）
- Stripe決済

### Phase 3（+1週間）
- Gemfile, requirements.txt, go.mod対応
- チーム機能
- 週次サマリーメール
- CSV/PDFエクスポート

## マーケ計画

### 初期（1-3ヶ月目）
- **SEO**: 「Node.js EOL 一覧」「Python バージョン サポート期限」「Ubuntu LTS 期限」
- **Qiita/Zenn**: 「あなたのプロジェクト、EOL大丈夫？」技術記事
- **Product Hunt**: 「Track EOL dates for your tech stack」
- **GitHub**: READMEバッジ「EOL Status: All Supported ✅」提供

### 中期（3-6ヶ月目）
- **開発者コミュニティ**: connpass, DevRel向けスポンサー
- **CI/CD連携**: GitHub Actionsでpush時にEOLチェック
- **セキュリティ監査対応**: 「EOL管理の証跡」としての付加価値

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 14, TailwindCSS, shadcn/ui, D3.js (タイムライン) |
| Backend | AWS Lambda (Node.js 20) |
| DB | DynamoDB |
| Auth | NextAuth.js (Email + GitHub OAuth) |
| EOL Data | endoflife.date API |
| Hosting | Vercel + AWS |
| CI/CD | GitHub Actions |
| 決済 | Stripe |
| 通知 | SES, Slack Webhook |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| endoflife.date API停止 | データ取得不能 | ローカルキャッシュ＋手動更新フォールバック |
| DepsFenceとの差別化認知 | 混乱 | DepsFenceは脆弱性・ライセンス、EOLTrackerはサポート期限に特化 |
| 大規模プロジェクトの依存関係が膨大 | 情報過多 | 主要技術（言語/FW/OS/DB）に絞る。ライブラリレベルはDepsFenceに任せる |
| GitHub OAuth審査 | 遅延 | public repos は認証不要。private repos対応はPhase 3 |

## 競合分析

| 競合 | 特徴 | EOLTrackerの優位性 |
|------|------|-------------------|
| endoflife.date（生データ） | Webサイトで手動確認 | プロジェクト単位の管理、アラート通知、ダッシュボード |
| Snyk | 脆弱性スキャン | EOL特化ではない。$25/月〜 |
| Renovate/Dependabot | 依存関係更新PR | バージョンアップ提案だがEOL期限の可視化はない |
| スプレッドシート管理 | 手動 | 自動更新なし、通知なし、漏れる |

## $20達成シナリオ

| シナリオ | Pro ($5) | Team ($12) | MRR |
|---------|---------|------------|-----|
| 最速 | 4人 | 0人 | $20 |
| 現実的（3ヶ月目） | 3人 | 1人 | $27 |
| 保守的（6ヶ月目） | 4人 | 2人 | $44 |

### 達成根拠
- EOL管理は全開発チームの課題だが、体系的にやっている所は少ない
- セキュリティ監査でEOL管理が指摘されるケースが増加
- endoflife.date APIが無料で使えるため、差別化はUX＋プロジェクト管理＋通知
- 「Node.js 18のEOL知ってた？」は開発者間で共感されやすいトピック

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0（SEO+技術記事主体） |
| ARPU | $6.75（Pro:Team = 3:1想定） |
| 粗利率 | 99.4%（$0.10/$33.75 @5ユーザー） |
| LTV（12ヶ月） | $81.00 |
| LTV/CAC | ∞（有機獲得前提） |
| 月間チャーン率 | 4%（開発プロジェクト継続中は必要） |
| 損益分岐ユーザー数 | 1人 |
