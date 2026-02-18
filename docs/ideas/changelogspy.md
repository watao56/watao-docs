# 🔍 ChangelogSpy — 依存SaaS/APIのBreaking Change検知

## 概要

Stripe、Shopify、Twilio、SendGrid等、自分のプロダクトが依存しているSaaS/APIのchangelog・リリースノート・deprecation noticeを自動監視し、Breaking Changeや重要な変更をSlack/メールで即時通知するサービス。「知らないうちにAPIが廃止されてサービスが止まった」を防ぐ。

## ターゲット

- **メイン**: SaaS開発者・CTOが1-3人の小規模スタートアップ
- **サブ**: フリーランスエンジニア（複数クライアントのSaaS依存を管理）
- **ペルソナ**: 佐藤さん（28歳）フリーランスエンジニア。5つのクライアント案件を抱え、それぞれStripe/SendGrid/Firebase等に依存。先月、Stripe APIのバージョン非推奨化に気づかず、クライアントの決済が3時間止まった。信頼と$500の損害。

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 3サービス監視、週1回チェック、メール通知 |
| Pro | $6/月 | 20サービス監視、日次チェック、Slack/Discord通知、Breaking Changeハイライト |
| Team | $15/月 | 無制限監視、リアルタイムチェック、チーム通知、カスタムRSSフィード |

## ユーザーフロー

1. サインアップ（GitHub OAuth）
2. 監視したいサービスを選択（Stripe, AWS, Firebase, Shopify等。200+のプリセット）
3. 使用中のAPIバージョンを登録（任意）
4. 通知先設定（Slack webhook / メール / Discord）
5. 毎日自動でchangelogページをクロール
6. Breaking Change / Deprecation / Security Fix を検知 → 即時通知
7. ダッシュボードで全依存サービスの変更履歴を一覧

## アーキテクチャ

```
[200+ SaaS Changelog URLs]
        ↓
[AWS Lambda (クローラー/パーサー)] ← EventBridge (日次トリガー)
        ↓
  [差分検知エンジン]
        ↓ (新規変更あり)
  [重要度分類: Breaking / Deprecation / Feature / Fix]
        ↓
  [Supabase PostgreSQL (変更履歴保存)]
        ↓
  [通知エンジン → Slack/Email/Discord]
        
[ユーザー] → [Next.js on Vercel] → [Supabase]
```

## DB設計

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  plan TEXT DEFAULT 'free',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 監視対象SaaSマスター
CREATE TABLE services (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL, -- 'Stripe', 'Firebase', etc.
  changelog_url TEXT NOT NULL,
  api_docs_url TEXT,
  icon_url TEXT,
  category TEXT -- payment, email, auth, database
);

-- ユーザーの監視設定
CREATE TABLE watchlist (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  service_id UUID REFERENCES services(id),
  api_version TEXT, -- ユーザーが使用中のバージョン
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, service_id)
);

-- 検知した変更
CREATE TABLE changes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  service_id UUID REFERENCES services(id),
  title TEXT NOT NULL,
  summary TEXT,
  severity TEXT, -- breaking, deprecation, security, feature, fix
  raw_content TEXT,
  source_url TEXT,
  published_at TIMESTAMPTZ,
  detected_at TIMESTAMPTZ DEFAULT NOW()
);

-- 通知設定
CREATE TABLE notification_channels (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  type TEXT NOT NULL,
  config JSONB NOT NULL,
  enabled BOOLEAN DEFAULT true
);

-- 通知履歴
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  change_id UUID REFERENCES changes(id),
  channel_id UUID REFERENCES notification_channels(id),
  sent_at TIMESTAMPTZ DEFAULT NOW()
);
```

## コスト見積もり

| 項目 | 月額 |
|------|------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| AWS Lambda (クローラー) | $0.30 (200サービス×日次×30日) |
| AWS Lambda (通知処理) | $0.10 |
| **合計** | **$0.40/月** (100ユーザー時) |

## MVPスコープ（2週間）

### Week 1
- GitHub OAuth認証
- 上位30サービスのchangelogクローラー実装（Stripe, AWS, Firebase, Vercel, Shopify, Twilio, SendGrid等）
- 差分検知ロジック（前回クロール結果との比較）
- 重要度分類（キーワードベース: "breaking", "deprecated", "removed", "security"）

### Week 2
- Slack/メール通知
- ダッシュボード（監視中サービス一覧、最近の変更）
- ウォッチリスト管理UI
- LP + Stripe決済

## マーケ計画

1. **SEO**: 「Stripe API 変更」「Firebase deprecated」等の検索に対応する記事
2. **Twitter/X**: API障害/変更のニュースに引用リプ → 「これを自動で検知するツール作りました」
3. **Dev.to/Zenn/Qiita**: 「依存SaaSのBreaking Changeで痛い目に遭った話」記事
4. **GitHub**: 人気OSSのdependency更新PRに関連してChangelogSpyを紹介
5. **Product Hunt**: 開発者向けツールとしてローンチ
6. **Indie Hackers**: ビルドログを公開

## 技術スタック

- **フロントエンド**: Next.js 14 + Tailwind CSS + shadcn/ui
- **バックエンド**: AWS Lambda (Node.js) — クローラー＆パーサー
- **DB**: Supabase (PostgreSQL)
- **認証**: Supabase Auth (GitHub OAuth)
- **決済**: Stripe
- **クローリング**: cheerio + node-fetch（軽量HTML解析）
- **通知**: Slack Webhook, AWS SES, Discord Webhook

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| Changelogページの構造変更 | 高 | サービスごとにパーサーを分離。RSSフィード優先、HTML解析はフォールバック |
| 偽陽性（重要でない変更をBreakingと誤判定） | 中 | キーワードベース＋コンテキスト解析。ユーザーフィードバックで改善 |
| SaaS公式がRSSを廃止 | 低 | ページ差分検知にフォールバック |
| 競合（Dependabot等） | 中 | Dependabotはコードレベル。ChangelogSpyはサービスレベル（API変更・機能廃止）で差別化 |

## 競合分析

| サービス | 月額 | 特徴 | ChangelogSpyの優位性 |
|----------|------|------|---------------------|
| Dependabot | 無料 | パッケージの脆弱性更新 | SaaS/APIレベルの変更監視（別レイヤー） |
| ChangeTower | $5〜 | 汎用ページ変更監視 | SaaS特化でBreaking Change自動分類 |
| RSS Reader | 無料 | フィード購読 | 自動フィルタリング＋重要度分類 |
| 手動チェック | 無料 | 各サービスのブログを巡回 | 200+サービスを自動監視、見落としゼロ |

## $20達成シナリオ

```
Month 1: 無料ユーザー80人（開発者コミュニティ経由）
Month 2: Pro 2人転換 = $12/月
Month 3: Pro 3人 + Team 1人 = $33/月 ✅
```

**必要有料ユーザー数: Pro 4人で$24達成（$20超え）**

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0（オーガニック） |
| ARPU | $6.00 |
| 粗利率 | 99.3%（$6.00 - $0.004/ユーザー） |
| LTV（12ヶ月） | $72 |
| LTV/CAC | ∞ |
| チャーン率（予測） | 2%/月（依存管理=継続必須） |
| Payback Period | 即時 |
