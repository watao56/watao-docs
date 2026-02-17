# 🔔 GitNudge - GitHub Issue/PR放置検知＋自動リマインダーBot

## 概要
GitHub Issue/PRの放置検知＋自動リマインダーBot。X日以上放置されたIssue/PRをSlack/Discordに通知し、OSSメンテナー・小規模チームの「放置問題」を解決する開発者向けSaaS。

## ターゲット
### プライマリ
- **OSSメンテナー**: 個人・小規模OSS（1-10名）
- **スタートアップ開発チーム**: 急成長で管理が追いつかない
- **フリーランス開発者**: 複数プロジェクトの並行管理
- **副業開発チーム**: 限られた時間での効率管理

### セカンダリ
- 中小企業の内製開発チーム
- GitHub中心の開発サイクルを持つ全組織
- コード品質・レビュー文化向上を目指すチーム

### ペイン
- **Issue/PR放置**: 重要な対応が埋もれて忘れられる
- **手動管理の限界**: GitHubの通知だけでは管理困難
- **レビュー遅延**: PRが放置されて開発フロー停滞
- **コントリビューター離脱**: 反応がないOSSへの失望

## 料金
- **フリープラン**: 1リポジトリまで無料
- **個人**: 月$3（5リポジトリまで）
- **チーム**: 月$8（20リポジトリ、チーム機能）
- **組織**: 月$20（無制限リポジトリ、カスタムルール）

## ユーザーフロー
1. **GitHub認証**: OAuth認証でリポジトリアクセス許可（30秒）
2. **リポジトリ選択**: 監視対象リポジトリを選択（1分）
3. **ルール設定**: 放置期間・通知先・除外条件設定（2分）
4. **通知チャンネル連携**: Slack/Discord Webhook設定（1分）
5. **自動監視開始**: 定期的にスキャン・通知開始

### 通知例
```
🚨 GitNudge Alert - myproject/awesome-app

📋 Stale Issues (3日以上):
• #42 "Add dark mode support" by @contributor (5 days ago)
• #38 "Fix memory leak in parser" by @user123 (7 days ago)

📬 Stale PRs (1日以上):
• #45 "Implement user authentication" by @dev456 (2 days ago)
• #41 "Update dependencies" by @maintainer (3 days ago)

💡 Suggested actions: Review, label, or close stale items
```

## アーキテクチャ
### フロントエンド
- **Dashboard**: Next.js + TypeScript + Tailwind CSS
- **GitHub OAuth**: NextAuth.js with GitHub provider

### バックエンド
- **API**: Node.js + Fastify
- **GitHub API**: Octokit.js（公式SDK）
- **DB**: PostgreSQL（リポジトリ、ルール、通知履歴）
- **Scheduler**: Node-cron（定期チェック）
- **Queue**: Redis + Bull（GitHub API制限対応）

### インフラ
- **App**: Railway（$5/月）
- **DB**: Supabase Free → Pro（$0 → $25）
- **Redis**: Upstash（$3/月）
- **Total**: $8-33/月（成長段階に応じて）

## DB設計
```sql
-- ユーザー
users (
  id, github_id, username, avatar_url, plan,
  access_token, created_at, updated_at
)

-- 監視リポジトリ
watched_repos (
  id, user_id, repo_name, repo_full_name,
  issue_threshold_days, pr_threshold_days,
  slack_webhook, discord_webhook, enabled
)

-- 通知ルール
notification_rules (
  id, repo_id, type, threshold_days,
  exclude_labels, exclude_assignees, enabled
)

-- 通知履歴
notifications (
  id, repo_id, issue_number, type,
  sent_at, status, response_data
)
```

## 技術スタック
- **Frontend**: Next.js 14, TypeScript, Tailwind CSS
- **Backend**: Node.js, Fastify, Prisma ORM
- **GitHub Integration**: Octokit.js, GitHub Webhooks
- **Database**: PostgreSQL, Redis
- **Notifications**: Slack API, Discord Webhooks
- **Auth**: NextAuth.js（GitHub OAuth）
- **Deployment**: Railway
- **Scheduling**: Node-cron, Bull Queue

## MVPスコープ（2週間）
### 必須機能
- GitHub OAuth認証
- リポジトリ選択・設定
- 基本的な放置検知ロジック
- Slack/Discord通知
- シンプルなダッシュボード
- 料金プラン管理（Stripe）

### Phase 2機能
- カスタム通知ルール
- チーム管理機能
- 詳細な分析・レポート
- GitHub Actions連携
- メール通知

## コスト見積もり
### 固定コスト（月額）
- Railway: $5
- Supabase: $0-25（成長に応じて）
- Upstash: $3
- **合計**: $8-33/月

### 変動コスト
- GitHub API: 無料（5,000 requests/hour）
- Slack/Discord API: 無料
- **実質変動コスト**: $0

## マーケ計画
### 顧客獲得
1. **GitHub**: 人気OSSリポジトリにissue提起・PR
2. **Dev.to/Zenn**: 「OSSメンテナンスの効率化」記事
3. **Twitter**: #OSS #GitHub #開発効率化 でコンテンツ発信
4. **Product Hunt**: GitHubツールとしてローンチ

### リテンション
- 放置問題解決による開発フロー改善実感
- OSSコントリビューター増加の実績
- チーム生産性向上の可視化

## 競合分析
### 既存サービス
1. **GitHub Notifications**: 標準機能だが管理困難
2. **GitHub Projects**: プロジェクト管理機能（放置検知なし）
3. **Dependabot**: 依存関係のみ、Issue/PR全般ではない
4. **CodeClimate**: コード品質（放置管理ではない）

### 差別化
- **放置特化**: この問題に完全特化
- **チャンネル統合**: Slack/Discordで集中管理
- **カスタムルール**: 柔軟な条件設定
- **開発フロー改善**: レビュー効率化にフォーカス

## リスク
### 技術リスク
- **GitHub API制限**: Rate limiting（5,000/hour）
  - **対策**: 効率的なAPI使用、キャッシュ活用
- **OAuth権限**: リポジトリアクセス権限の管理
  - **対策**: 必要最小限の権限要求

### ビジネスリスク
- **競合参入**: GitHubが同等機能を標準化
  - **対策**: カスタム性・外部連携で差別化
- **市場規模限定**: GitHub利用者に限定
  - **対策**: GitLab等の他プラットフォーム拡張

### 運用リスク
- **通知スパム**: 過度な通知によるユーザー離脱
  - **対策**: カスタマイズ性・頻度制限

## $20達成シナリオ
### 想定単価・歩留まり
- 個人（$3）: 60%
- チーム（$8）: 30%
- 組織（$20）: 10%
- **平均単価**: $6.2

### 必要ユーザー数
- **$20 ÷ $6.2 = 3.2人**
- **固定費$33込み**: $53必要 ÷ $6.2 = 8.5人
- **安全マージン込み**: **10人**

### 獲得計画
- 1ヶ月目: 3人（個人ネットワーク・直接営業）
- 2ヶ月目: 8人（記事・GitHub活動）
- 3ヶ月目: 15人（Product Hunt・口コミ）

### 根拠
- **TAM**: GitHub積極利用者約50万人（国内）
- **SAM**: OSSメンテナー・小規模チーム約10万人
- **SOM**: 年0.01%獲得で10人（$20達成）

## ユニットエコノミクス
### 収益
- 個人: $3/月 × 60% = $1.8
- チーム: $8/月 × 30% = $2.4
- 組織: $20/月 × 10% = $2.0
- **合計ARPU**: $6.2/月

### コスト（10ユーザー想定）
- **固定コスト**: $33/月 ÷ 10人 = $3.3/user/month
- **変動コスト**: $0/user/month（API無料）
- **総コスト**: $3.3/user/month
- **粗利**: $2.9/user/month（**47%**）

### スケール後（100ユーザー想定）
- **固定コスト**: $33/月 ÷ 100人 = $0.33/user/month
- **粗利**: $5.87/user/month（**95%**）

---

## 設計書v1 完成（レビュー待ち）