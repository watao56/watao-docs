# 🔔 GitNudge - GitHub放置検知 & 自動リマインダー

## 概要
GitHub Issue・PRの放置を自動検知し、設定日数経過で担当者・チームにSlack/Discord/Teams自動通知。「あのPR放置されてる」「Issueレスポンスなし」の解決で、開発チームの生産性とコミュニケーション品質を向上。

## ターゲット
### プライマリ
- **スタートアップCTO・リードエンジニア（5-50人チーム）**: 開発速度とコードレビュー品質の両立
- **アジャイルチームリーダー**: スプリント内での確実なタスク完了
- **オープンソース メンテナー**: 大量Issue・PRの効率的管理

### セカンダリ
- フリーランス開発チーム（複数プロジェクト並行）
- 大企業の開発部門（管理職・PMレベル）
- 学生・新人エンジニア（フォローアップ強化目的）

### ペイン
- **放置の見える化不足**: 「あれどうなった？」の確認コスト
- **レビュー遅延**: PRが放置され、デプロイが遅れる
- **コミュニケーションコスト**: 進捗確認のための個別メッション
- **品質低下**: 急いでレビューして見落としが発生

### 解決価値
- **開発速度**: PR処理時間50%短縮（平均3日→1.5日）
- **品質向上**: 放置による急ぎレビューを防止
- **ストレス軽減**: 手動フォローアップ不要

## 料金
- **オープンソース**: 無料（コミュニティ貢献）
- **チームプラン**: 月$12（5人まで、無制限リポジトリ）
- **ビジネス**: 月$25（20人まで、分析ダッシュボード、SLA監視）
- **エンタープライズ**: 月$60（無制限、オンプレ、SSO、監査ログ）

## ユーザーフロー
1. **GitHub連携**: OAuth認証でリポジトリアクセス許可（30秒）
2. **チーム設定**: Slack/Discord Webhook設定（1分）
3. **ルール設定**: 放置判定日数・通知対象・例外設定（2分）
4. **監視開始**: バックグラウンドで自動監視開始
5. **自動通知**: 条件満たしたら即座にチャット通知
6. **アクション**: 通知から直接GitHub該当ページへ

### 通知例
```
🚨 PR放置アラート 🚨
リポジトリ: myapp/frontend
PR: #123 「ユーザー認証機能追加」
担当者: @john_doe
放置期間: 3日
アクション: [レビューする] [アサイン変更] [延期設定]
```

## アーキテクチャ
### フロントエンド
- **Dashboard**: Next.js + TypeScript + Chart.js
- **Setup Wizard**: 簡単3ステップ設定UI
- **Analytics**: チーム生産性分析ダッシュボード

### バックエンド
- **API**: Node.js + Express + TypeScript
- **DB**: PostgreSQL（設定・履歴・分析データ）
- **GitHub API**: Webhook + REST API
- **通知**: Slack/Discord/Teams Webhooks
- **Scheduler**: Cron Jobs（定期チェック・リマインダー）

### インフラ
#### Phase 0（0-5チーム、完全無料）
- **Vercel Hobby**: $0/月（フロントエンド）
- **Railway Starter**: $0/月（バックエンド）
- **Supabase Free**: $0/月（DB 500MB）
- **合計**: **$0/月**

#### Phase 1（5-50チーム）
- **Vercel Pro**: $20/月
- **Railway Pro**: $20/月
- **Supabase Pro**: $25/月
- **合計**: $65/月

## DB設計
```sql
-- チーム・組織
teams (
  id, name, github_org, slack_webhook,
  discord_webhook, plan_type, created_at
)

-- リポジトリ監視設定
repositories (
  id, team_id, github_repo_id, name,
  pr_alert_days, issue_alert_days,
  exclude_drafts, exclude_labels, is_active
)

-- アラートルール
alert_rules (
  id, repo_id, type, condition,
  alert_after_days, notification_channels,
  assignee_ping, team_ping
)

-- 放置検知履歴
nudge_history (
  id, repo_id, github_item_id, type,
  assignee, detected_at, notified_at,
  resolved_at, response_time
)

-- チーム分析
team_analytics (
  id, team_id, date, avg_pr_time,
  avg_issue_time, nudge_count, resolved_count
)
```

## 技術スタック
- **Frontend**: Next.js 14, TypeScript, Tailwind CSS, Chart.js
- **Backend**: Node.js, Express.js, Prisma ORM
- **Database**: PostgreSQL
- **External APIs**: GitHub API, Slack API, Discord API
- **Deployment**: Vercel + Railway
- **Auth**: NextAuth.js（GitHub OAuth必須）
- **Queue**: Bull (Redis) for background jobs

## MVPスコープ（2週間）
### 必須機能
- GitHub OAuth認証・リポジトリ選択
- Slack Webhook設定・通知送信
- PR/Issue放置検知（3日・7日・14日設定）
- 基本ダッシュボード（放置一覧・設定管理）
- チーム招待・権限管理
- Stripe決済（チーム・ビジネスプラン）

### Phase 1機能（MVP）
- 1通知チャンネル（Slack）
- 基本放置検知
- シンプル設定UI

### Phase 2機能（差別化）
- **複数チャンネル**: Discord, Teams, メール通知
- **スマート検知**: ドラフト、WIP、特定ラベル除外
- **分析ダッシュボード**: チーム生産性可視化
- **エスカレーション**: 2次・3次通知設定

## コスト見積もり
### Phase 0（完全無料、0-5チーム）
- **固定コスト**: $0/月
- **変動コスト**: $0/月（無料枠内）
- **粗利率**: 100%

### Phase 1（50チーム想定）
- **固定コスト**: $65/月
- **変動コスト**: GitHub API制限内（無料）
- **総コスト**: $65/月
- **収益**: 50チーム × $15平均 = $750/月
- **粗利**: $685/月（91%）

### 変動コストは軽微
- GitHub API: 無料枠5,000リクエスト/時間で十分
- Webhook送信: 無料
- 主要コストはインフラのみ

## マーケ計画
### 開発者特化チャネル
1. **GitHub Marketplace**: GitHub公式アプリストアで発見可能性
2. **Dev.to/Zenn**: 「チーム開発効率化」記事投稿
3. **Hacker News**: 「Show HN: GitHub Nudger」で初期ユーザー
4. **Reddit**: r/programming, r/webdev での体験談投稿
5. **Product Hunt**: 開発者ツールカテゴリでローンチ

### コミュニティ・口コミ戦略
- **オープンソース無料**: OSS採用→有料チームに流入
- **GitHub統合**: リポジトリのREADMEにバッジ表示
- **Slack App Store**: Slack公式ディレクトリ掲載

### エンタープライズ営業
- **技術カンファレンス**: DevOps・アジャイル関連イベント出展
- **ウェビナー**: 「チーム開発生産性向上」テーマ
- **パートナー連携**: GitHub、Slack、Atlassianとの協業

## 競合分析
### 既存ソリューション
1. **GitHub Projects**: 基本的なトラッキング、通知機能限定
2. **Linear**: 高機能だが月$8/人、GitHub特化でない
3. **Jira**: 過剰機能で小チーム向けでない、月$7/人
4. **Zapier統合**: 手動設定複雑、$20/月～

### 差別化ポイント
- **GitHub特化**: Issue・PR専用で無駄な機能なし
- **簡単設定**: 3分で完了 vs 競合の1時間設定
- **圧倒的低価格**: $12/5人 vs Linear $40/5人
- **チーム最適化**: 個人でなくチーム単位の最適化

## リスク分析
### 技術リスク（低）
- **GitHub API制限**: 無料枠5,000/時は十分（50チーム対応可能）
- **Webhook信頼性**: GitHub Webhookは高信頼性
- **拡張性**: 段階的インフラで対応

### ビジネスリスク（中）
- **GitHub機能追加**: GitHub自体が同機能提供の可能性
- **大手競合**: Microsoft/Atlassian参入リスク
- **対策**: 早期シェア確保、細かいニーズ特化

### 市場リスク（低）
- **開発チーム限定**: TAMは限定的だが十分
- **経済変動**: 開発効率化は不況でも需要継続

## $20達成シナリオ
### 料金・ユーザー構成
- **チーム$12**: 60%
- **ビジネス$25**: 35%
- **エンタープライズ$60**: 5%
- **加重平均**: $18.3/月

### 必要チーム数
- **$20 ÷ $18.3 = 1.09チーム**
- **Phase 0無料期間**: **2チームで即$20達成**
- **安全マージン**: 3チーム（$54.9/月）

### 獲得根拠
- **TAM**: 国内開発チーム約10万（大企業含む）
- **SAM**: 5-50人規模の開発チーム約2万
- **SOM**: アーリーアダプター1%→200チーム
- **現実的目標**: 初年度50チーム→十分なバッファ

### 3ヶ月計画
- **1ヶ月目**: 5チーム（Product Hunt経由）
- **2ヶ月目**: 15チーム（コミュニティ拡散）
- **3ヶ月目**: 30チーム（紹介・口コミ）

## ユニットエコノミクス
### Phase 0（完全無料運用）
- **固定コスト**: $0/月
- **変動コスト**: $0/month
- **ARPU**: $18.3/month
- **粗利**: $18.3/team（100%）

### Phase 1（50チーム運用）
- **固定コスト**: $65 ÷ 50チーム = $1.3/team/month
- **変動コスト**: $0.1/team/month（通信費等）
- **総コスト**: $1.4/team/month
- **粗利**: $16.9/team/month（92%）

### 継続性・LTV
- **平均継続期間**: 18ヶ月（開発ツールは継続率高）
- **LTV**: $18.3 × 18 = $329.4
- **CAC**: $30（コミュニティ・口コミ中心）
- **LTV/CAC**: 10.98（非常に健全）

## 成長戦略
### フリーミアム戦略
- **OSS無料**: コミュニティ貢献で認知拡大
- **有料転換**: チームメンバー増→自然にアップグレード

### プロダクトled成長
- **GitHub統合**: リポジトリバッジで認知拡大
- **紹介機能**: 他チームへの簡単招待

### 機能拡張（将来）
- **コード品質監視**: ESLint/TSLint結果監視
- **デプロイ監視**: CI/CD失敗通知
- **総合開発ツール**: GitNudgeを入口とした統合プラットフォーム

---

## 設計書完成
**GitNudge: GitHub放置をゼロにする自動リマインダーで、開発チーム生産性を劇的向上**