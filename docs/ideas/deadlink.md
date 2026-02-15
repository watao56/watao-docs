# DeadLink — 設計書

## 概要
ブログ・Webサイトのリンク切れを定期的に検知し、通知するSaaSサービス。

---

## 1. プロダクト定義

### 解決する課題
- ブログの外部リンクは時間とともに切れる（サービス終了、URL変更、ドメイン失効）
- リンク切れはSEO評価を下げ、読者体験を損なう
- 手動チェックは非現実的（記事100本 × リンク5本 = 500リンク）
- WordPress以外（Hugo, Astro, Zenn, はてなブログ）に対応した監視ツールがない

### ターゲットユーザー
- **メイン**: 記事50本以上の中堅〜ベテランブロガー・技術ブロガー
- **サブ**: 小規模メディア運営者、ポートフォリオサイト管理者

### 料金プラン

| | Free | Pro ($5/月) |
|---|---|---|
| サイト数 | 1 | 3 |
| チェック頻度 | 週1回 | 毎日 |
| 通知 | メールのみ | メール + Discord + Slack |
| レポート | 直近1回分 | 過去30日分の履歴 |
| ページ上限/サイト | 100ページ | 500ページ |

---

## 2. ユーザーフロー

```
[LP] URL入力で即スキャン（登録不要）
  ↓
[結果表示] リンク切れ一覧 + 件数
  ↓
[CTA] 「定期監視するなら無料登録」
  ↓
[登録] メール or GitHub OAuth
  ↓
[ダッシュボード] サイト管理 + 結果一覧
  ↓
[通知設定] メール / Discord Webhook / Slack Webhook
  ↓
[Pro誘導] サイト追加 or 毎日チェックしたい時
```

### LP即スキャン（最重要機能）
- 登録なしでURLを入れたら30秒以内に結果を返す
- 「あなたのサイト、リンク切れ○件あります」のインパクトで登録に繋げる
- この体験がそのまま口コミ素材になる

---

## 3. システムアーキテクチャ

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│   Vercel     │     │  API Gateway  │     │   Lambda     │
│  Next.js     │────▶│  + Lambda     │────▶│  Crawler     │
│  (Frontend)  │     │  (API)        │     │  (Worker)    │
└─────────────┘     └──────────────┘     └──────────────┘
                           │                      │
                           ▼                      ▼
                    ┌──────────────┐     ┌──────────────┐
                    │  Supabase    │     │  DynamoDB    │
                    │  (Auth +     │     │  (Scan       │
                    │   Users)     │     │   Results)   │
                    └──────────────┘     └──────────────┘
                                                 │
                                                 ▼
                                        ┌──────────────┐
                                        │  EventBridge │
                                        │  (Scheduler) │
                                        └──────────────┘
                                                 │
                                                 ▼
                                        ┌──────────────┐
                                        │  SES / SNS   │
                                        │  (通知)       │
                                        └──────────────┘
```

---

## 4. コンポーネント詳細

### 4.1 Frontend (Vercel + Next.js)

**ページ構成**
- `/` — LP + 即スキャンフォーム
- `/scan/:id` — スキャン結果ページ（シェア可能）
- `/dashboard` — ログイン後ダッシュボード
- `/dashboard/sites/:id` — サイト詳細 + リンク切れ一覧
- `/settings` — 通知設定、プラン管理
- `/pricing` — 料金ページ

**技術**
- Next.js 15 (App Router)
- Tailwind CSS
- Supabase Auth (GitHub OAuth + Magic Link)
- Stripe (決済) ※ 初期はBuyMeACoffeeでも可

### 4.2 API (API Gateway + Lambda)

**エンドポイント**

```
POST /api/scan/instant    — LP即スキャン（非認証、レート制限あり）
POST /api/sites           — サイト登録
GET  /api/sites           — サイト一覧
GET  /api/sites/:id       — サイト詳細
DELETE /api/sites/:id     — サイト削除
GET  /api/sites/:id/reports        — レポート一覧
GET  /api/sites/:id/reports/:rid   — レポート詳細
PUT  /api/settings/notifications   — 通知設定更新
```

**即スキャンのレート制限**
- IP単位: 3回/日
- 同一URL: 1回/時間（キャッシュ返却）

### 4.3 Crawler (Lambda Worker)

**処理フロー**
```
1. 対象サイトのURLを受け取る
2. トップページをfetch → HTML解析 → 内部リンク抽出
3. 各内部ページをfetch → HTML解析 → 全リンク（内部+外部）抽出
4. 全外部リンクに HEAD リクエスト（タイムアウト10秒）
5. ステータスコード記録:
   - 2xx → OK
   - 3xx → リダイレクト（警告として記録）
   - 4xx/5xx → リンク切れ
   - タイムアウト → リンク切れ（要確認）
6. 結果をDynamoDBに保存
7. リンク切れがあれば通知発火
```

**制約・配慮**
- 同一ドメインへのリクエスト間隔: 500ms（礼儀正しいクロール）
- User-Agent: `DeadLinkBot/1.0 (+https://deadlink.app/about)`
- robots.txt を尊重
- Lambda タイムアウト: 5分（100ページ×5リンク=500リクエストに十分）
- 1回のスキャンが5分超えそうなら分割実行（SQSでキューイング）

### 4.4 データベース設計

**Supabase (PostgreSQL) — ユーザー系**

```sql
-- ユーザー（Supabase Authが管理）
-- profiles テーブルで追加情報
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  plan TEXT DEFAULT 'free',  -- 'free' | 'pro'
  stripe_customer_id TEXT,
  notification_email BOOLEAN DEFAULT true,
  discord_webhook TEXT,
  slack_webhook TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- サイト
CREATE TABLE sites (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id),
  url TEXT NOT NULL,
  name TEXT,
  page_limit INT DEFAULT 100,
  check_interval TEXT DEFAULT 'weekly',  -- 'weekly' | 'daily'
  last_checked_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

**DynamoDB — スキャン結果（大量書き込み向き）**

```
Table: scan_reports
  PK: site_id (String)
  SK: scanned_at (String, ISO8601)
  Attributes:
    - total_links: Number
    - broken_links: Number
    - redirected_links: Number
    - status: String ('completed' | 'running' | 'failed')
    - duration_ms: Number

Table: scan_results
  PK: report_id (String)
  SK: link_url (String)
  Attributes:
    - source_page: String (どのページにあったか)
    - status_code: Number
    - status: String ('ok' | 'broken' | 'redirect' | 'timeout')
    - redirect_to: String (3xxの場合)
    - anchor_text: String
    - checked_at: String
```

### 4.5 スケジューラー (EventBridge)

```
- Free: 毎週日曜 AM3:00 JST に対象サイトをキューに投入
- Pro:  毎日 AM3:00 JST に対象サイトをキューに投入
- SQS経由でCrawler Lambdaを起動（同時実行数制御）
```

### 4.6 通知

**メール (SES)**
```
件名: [DeadLink] example.com で3件のリンク切れを検出
本文:
  - https://example.com/post-1 → リンク先: https://dead-service.com (404)
  - https://example.com/post-2 → リンク先: https://old-api.com/docs (503)
  - https://example.com/post-3 → リンク先: https://deleted.io (接続タイムアウト)
  
  詳細: https://deadlink.app/dashboard/sites/xxx/reports/yyy
```

**Discord / Slack (Webhook)**
- Embed形式で見やすく表示
- リンク切れ0件の時は通知しない（ノイズ防止）

---

## 5. インフラコスト見積もり

### 無料ユーザー100人 + Proユーザー5人の場合

| サービス | 用途 | 月額 |
|---|---|---|
| Vercel | フロントエンド | $0 (Hobby) |
| Supabase | Auth + DB | $0 (Free tier) |
| Lambda | API + Crawler | $0 (無料枠: 月100万リクエスト) |
| API Gateway | API | $0 (無料枠: 月100万リクエスト) |
| DynamoDB | スキャン結果 | $0 (無料枠: 25GB + 25 WCU/RCU) |
| EventBridge | スケジューラー | $0 |
| SES | メール通知 | $0 (月1000通無料) |
| SQS | ジョブキュー | $0 (無料枠: 月100万リクエスト) |
| ドメイン | deadlink.app 等 | ~$12/年 ≒ $1/月 |
| **合計** | | **~$1/月** |

### 収入
- Pro 5人 × $5 = $25/月
- **利益: $24/月** ✅ 目標$20達成

### スケール時（ユーザー1000人）
- Lambda実行数増加: 推定 $3-5/月
- DynamoDB: 推定 $2-3/月
- それでも $10/月以内に収まる

---

## 6. MVP スコープ（v1.0）

### Phase 1: MVP（2週間）
- [ ] LP + 即スキャンフォーム
- [ ] Crawler Lambda（基本クロール + リンクチェック）
- [ ] 結果表示ページ（シェア可能URL付き）
- [ ] Supabase Auth（GitHub OAuth）
- [ ] ダッシュボード（サイト登録 + 結果一覧）
- [ ] メール通知（SES）
- [ ] EventBridge定期実行

### Phase 2: 課金+通知拡張（1週間）
- [ ] Stripe連携（Pro課金）
- [ ] Discord / Slack Webhook通知
- [ ] レポート履歴表示

### Phase 3: 成長機能（随時）
- [ ] スキャン結果のOGP画像自動生成（シェア用）
- [ ] リンク切れ修正候補の提示（Internet Archive連携）
- [ ] WordPress REST API連携（記事内リンク直接編集）
- [ ] チーム機能

---

## 7. 周知・マーケティング計画

### ローンチ前
1. LP公開 + 即スキャン機能だけ先行リリース
2. 自分のブログをスキャンした結果をスクショ

### ローンチ（Day 1）
3. Zenn記事: 「ブログのリンク切れを自動検知するサービスを作った話」
   - 技術スタック解説 + 開発裏話 → 技術者に刺さる
4. X投稿: 結果スクショ + URL「無料でスキャンできます」

### ローンチ後（Week 2-4）
5. はてなブログ界隈で告知（はてブされやすい）
6. 「SEO リンク切れ」でSEO記事を書いてLP流入
7. Product Hunt Japan 等に掲載

### 継続
8. ユーザーのスキャン結果ページ自体がSEO資産になる
9. 「リンク切れ チェック 無料」のロングテールSEO

---

## 8. 技術スタック一覧

| レイヤー | 技術 | 理由 |
|---|---|---|
| Frontend | Next.js 15 + Tailwind | 高速開発、Vercel無料 |
| Auth | Supabase Auth | 無料、GitHub OAuth簡単 |
| User DB | Supabase PostgreSQL | Auth統合、無料 |
| Scan DB | DynamoDB | 大量書き込み向き、無料枠大 |
| Crawler | Lambda (Node.js) | サーバーレス、無料枠大 |
| HTML解析 | cheerio | 軽量、Lambda向き |
| Queue | SQS | Lambda統合簡単 |
| Scheduler | EventBridge | cron簡単、無料 |
| Mail | SES | 安い、信頼性高い |
| Payment | Stripe | 業界標準 |
| Hosting | Vercel | Next.js最適、無料 |
| Domain | Route53 or Cloudflare | 好みで |

---

## 9. リスクと対策

| リスク | 対策 |
|---|---|
| クロール先からブロックされる | User-Agent明示、間隔500ms、robots.txt尊重 |
| 大規模サイトでLambdaタイムアウト | ページ上限設定 + SQS分割実行 |
| 即スキャンの悪用（DDoSツール化） | IPレート制限 + CAPTCHA（閾値超え時） |
| 誤検知（一時的な503等） | 2回連続で切れた場合のみ「切れ」判定 |
| 無料ユーザーばかりで課金されない | 無料は週1 + 結果1回分のみ。痛みを感じさせてから課金導線 |
