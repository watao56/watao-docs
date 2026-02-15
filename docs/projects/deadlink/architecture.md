# アーキテクチャ

## システム構成図

```
┌─────────────┐     ┌──────────────────────────┐
│  Next.js     │     │  Next.js API Routes      │
│  Frontend    │────▶│  (scan, sites, reports)  │
│  (Vercel)    │     │  + Crawler Logic         │
└─────────────┘     └──────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
            ┌──────────────┐   ┌──────────────┐
            │  Supabase    │   │  DynamoDB    │
            │  Auth +      │   │  scan_reports│
            │  profiles/   │   │  scan_results│
            │  sites       │   │  (AWS)       │
            └──────────────┘   └──────────────┘
```

## 現在の構成

- **フロントエンド**: Next.js 15 (App Router) + Tailwind CSS
- **認証**: Supabase Auth (GitHub OAuth + Magic Link)
- **ユーザーDB**: Supabase PostgreSQL (profiles, sites)
- **スキャンDB**: AWS DynamoDB (deadlink_scan_reports, deadlink_scan_results)
- **クローラー**: Next.js API Routes内で直接実行
- **ホスティング**: Vercel (予定)

## 将来のアーキテクチャ（Phase 2+）

- **クローラー** → Lambda (SQSトリガー)
- **スケジューラー** → EventBridge + Lambda
- **通知** → SES (メール) + Webhook (Discord/Slack)
- **API** → API Gateway + Lambda

## データベース設計

### Supabase (PostgreSQL) — ユーザー・サイト管理

```sql
-- ユーザープロフィール
profiles (id, plan, stripe_customer_id, notification_email, discord_webhook, slack_webhook)

-- 登録サイト
sites (id, user_id, url, name, page_limit, check_interval, last_checked_at)
```

### DynamoDB (AWS ap-northeast-1) — スキャン結果

```
Table: deadlink_scan_reports
  PK: site_id (String)
  SK: scanned_at (String, ISO8601)
  GSI: report_id-index (PK: report_id)
  Attributes: report_id, url, total_links, broken_links, redirected_links, status, duration_ms

Table: deadlink_scan_results
  PK: report_id (String)
  SK: link_url (String)
  Attributes: source_page, status_code, status, redirect_to, anchor_text, checked_at
```

**プロビジョニング**: 各テーブル 5 RCU / 5 WCU（無料枠内）

## クローラーの仕組み

1. 対象サイトのトップページをfetch → HTML解析 → 内部リンク抽出
2. 各内部ページを巡回 → 全リンク（内部+外部）抽出
3. 外部リンクにHEADリクエスト（タイムアウト10秒）
4. ステータスコード判定:
   - 2xx → OK
   - 3xx → リダイレクト（警告）
   - 4xx/5xx → リンク切れ
   - タイムアウト → リンク切れ（要確認）

### クロール配慮
- 同一ドメインへのリクエスト間隔: 300-500ms
- User-Agent: `DeadLinkBot/1.0`
- robots.txt 尊重（未実装、Phase 2で対応予定）
