# 開発ログ

## 2026-02-15: Phase 1 MVP 開発

### 実施内容

1. **Next.js フロントエンド構築**
   - LP + 即スキャンフォーム
   - ログイン画面 (GitHub OAuth + Magic Link)
   - ダッシュボード (サイト登録・一覧)
   - サイト詳細 (レポート一覧)
   - 通知設定画面
   - 料金ページ
   - スキャン結果ページ (シェア可能)

2. **クローラー実装**
   - cheerio によるHTML解析
   - 内部リンク巡回 + 外部リンクHEADチェック
   - Next.js API Routes で直接実行（MVP）
   - Lambda用の独立実装も作成

3. **API Routes**
   - `POST /api/scan/instant` - 即スキャン（非認証、IP制限3回/日）
   - `GET /api/scan/[id]` - レポート取得
   - `GET /api/sites/[id]/reports` - サイトのレポート一覧
   - `POST /api/sites/[id]/scan` - 手動スキャン実行

4. **Supabase設定**
   - テーブル定義SQL作成 (profiles, sites, scan_reports, scan_results)
   - RLSポリシー設定
   - 自動プロフィール作成トリガー

### 設計変更

- **DynamoDB → Supabase PostgreSQL**: AWS IAMユーザー `OpenClaw` に権限がなく、DynamoDB/SQS/Lambda等のリソース作成ができなかった。MVPではSupabase PostgreSQLで全データを管理する形に変更。
- **Lambda → Next.js API Routes**: 同様の理由でLambdaデプロイができないため、クローラーロジックをNext.js API Routes内で直接実行。将来的にAWS権限が付与されたらLambda+SQSに移行予定。

### 使用サービス・プラン情報

| サービス | プラン | 月額 | 備考 |
|---|---|---|---|
| Supabase | Free | $0 | Auth + PostgreSQL |
| Vercel | Hobby | $0 | Next.jsホスティング（未デプロイ） |
| AWS | Free Tier | $0 | IAM権限未設定のため未使用 |

## 2026-02-15: DynamoDB移行

### 実施内容

- **スキャン結果の保存先をSupabase PostgreSQL → AWS DynamoDBに変更**
- DynamoDBテーブル作成（ap-northeast-1リージョン）:
  - `deadlink_scan_reports` (PK: site_id, SK: scanned_at, GSI: report_id-index)
  - `deadlink_scan_results` (PK: report_id, SK: link_url)
  - 各5 RCU/WCU（無料枠内）
- `src/lib/dynamodb.ts` を @aws-sdk/lib-dynamodb ベースに書き換え
- npm install @aws-sdk/client-dynamodb @aws-sdk/lib-dynamodb
- ユーザー管理・認証はSupabaseのまま（変更なし）
- ビルド確認済み（`npm run build` 成功）

### 次のフェーズで必要なもの

1. **Vercel アカウント** - フロントエンドデプロイ用
2. **Stripe アカウント** - Pro課金 ($5/月) 実装
3. **GitHub OAuth App** - Supabase Auth用のClient ID / Secret
4. **ドメイン** - deadlink.app 等の取得

### TODO (Phase 2)

- [ ] Supabase SQL Editorでテーブル作成を実行
- [ ] GitHub OAuth App 設定
- [ ] Vercel デプロイ（AWS環境変数設定必要）
- [ ] Stripe 連携
- [ ] Discord/Slack Webhook通知
- [ ] Lambda + SQS移行（クローラー分離）
- [ ] SES設定 → メール通知
- [ ] EventBridge定期実行
