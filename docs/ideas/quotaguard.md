# 💸 QuotaGuard — クラウド請求爆発防止アラート

## 概要

AWS・GCP・Vercel・Cloudflareなどのクラウドサービスの請求額を毎日チェックし、設定した予算を超えそうになったら即座にSlack/メール/LINEで通知するサービス。「朝起きたらAWSから$3,000の請求が来てた」を二度と起こさない。

## ターゲット

- **メイン**: 個人開発者・スタートアップ（月$50〜$500のクラウド利用）
- **サブ**: 小規模Web制作会社（複数クライアントのAWSアカウント管理）
- **ペルソナ**: 個人開発者の田中さん（32歳）。Lambda + DynamoDBで副業SaaSを運営。先月、DDoSでCloudFrontの転送量が跳ね上がり$200の想定外請求。AWS Budgetsは設定が面倒で放置していた。

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 1アカウント、1日1回チェック、メール通知のみ |
| Pro | $5/月 | 5アカウント、1時間ごとチェック、Slack/LINE/Discord通知、日次コストレポート |
| Team | $12/月 | 20アカウント、15分ごとチェック、チームSlack通知、異常検知AI |

## ユーザーフロー

1. サインアップ（GitHub OAuth）
2. クラウドプロバイダーを接続（AWS: IAMロール、GCP: サービスアカウント、Vercel: APIトークン）
3. 予算しきい値を設定（例: 月$100、日$10）
4. 通知先を設定（Slack webhook / メール / LINE Notify）
5. 毎日/毎時の自動チェック開始
6. しきい値超過 or 異常な増加パターン検知 → 即時通知
7. ダッシュボードで日次/月次コスト推移を確認

## アーキテクチャ

```
[ユーザー] → [Next.js on Vercel (フロントエンド)]
                    ↓
            [API Routes (認証・設定管理)]
                    ↓
            [Supabase (PostgreSQL + Auth)]
                    ↓
[AWS EventBridge] → [Lambda (コストチェッカー)]
                    ↓
            [AWS Cost Explorer API / GCP Billing API / Vercel API]
                    ↓
            [SNS → Slack/メール/LINE通知]
```

## DB設計

```sql
-- ユーザー
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  github_id TEXT,
  plan TEXT DEFAULT 'free',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- クラウドアカウント接続
CREATE TABLE cloud_accounts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  provider TEXT NOT NULL, -- aws, gcp, vercel, cloudflare
  credentials_encrypted TEXT NOT NULL,
  display_name TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 予算設定
CREATE TABLE budgets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  cloud_account_id UUID REFERENCES cloud_accounts(id),
  monthly_limit DECIMAL,
  daily_limit DECIMAL,
  alert_at_percent INTEGER DEFAULT 80
);

-- コスト履歴
CREATE TABLE cost_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  cloud_account_id UUID REFERENCES cloud_accounts(id),
  date DATE NOT NULL,
  amount DECIMAL NOT NULL,
  currency TEXT DEFAULT 'USD',
  breakdown JSONB, -- サービス別内訳
  recorded_at TIMESTAMPTZ DEFAULT NOW()
);

-- 通知設定
CREATE TABLE notification_channels (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  type TEXT NOT NULL, -- slack, email, line, discord
  config JSONB NOT NULL,
  enabled BOOLEAN DEFAULT true
);

-- 通知履歴
CREATE TABLE alerts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  cloud_account_id UUID REFERENCES cloud_accounts(id),
  type TEXT, -- threshold, anomaly, daily_report
  message TEXT,
  sent_at TIMESTAMPTZ DEFAULT NOW()
);
```

## コスト見積もり

| 項目 | 月額 |
|------|------|
| Vercel (フロントエンド) | $0 (Hobby) |
| Supabase (DB + Auth) | $0 (Free tier) |
| AWS Lambda (コストチェック) | $0.50 (100ユーザー×24回/日) |
| AWS Cost Explorer API | $0.01/リクエスト → 月$3 (100ユーザー) |
| SNS (通知送信) | $0.10 |
| **合計** | **$3.60/月** (100ユーザー時) |

## MVPスコープ（2週間）

### Week 1
- GitHub OAuth認証
- AWSアカウント接続（IAMロール作成ガイド付き）
- 予算しきい値設定UI
- Lambda: 日次コストチェック実装

### Week 2
- Slack/メール通知
- ダッシュボード（月次コスト推移グラフ）
- 日次コストレポートメール
- LP + Stripe課金

## マーケ計画

1. **SEO**: 「AWS 請求 高い」「クラウド コスト 管理」で記事量産
2. **Twitter/X**: 「AWS請求爆発」体験談に引用リプ → プロダクト紹介
3. **Reddit/HN**: r/aws, r/devops でShow HN投稿
4. **Qiita/Zenn**: 「AWSの請求で泣かないための3つの対策」→ 自然にQuotaGuard紹介
5. **Product Hunt**: ローンチ
6. **口コミ**: 無料プランで使い始め → 予算超過を防げた体験 → Pro転換

## 技術スタック

- **フロントエンド**: Next.js 14 (App Router) + Tailwind CSS + shadcn/ui
- **バックエンド**: Next.js API Routes + AWS Lambda (Python)
- **DB**: Supabase (PostgreSQL)
- **認証**: Supabase Auth (GitHub OAuth)
- **決済**: Stripe
- **通知**: AWS SNS, Slack Webhook, LINE Notify API
- **監視**: Vercel Analytics

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| AWS Budgets（無料）で十分と思われる | 高 | AWS Budgets比較表を作成。マルチクラウド対応・異常検知・簡単設定を差別化ポイントに |
| IAMロール設定のハードル | 中 | ワンクリックCloudFormationテンプレート提供 |
| Cost Explorer APIコスト | 中 | キャッシュ戦略で呼び出し回数最小化 |
| セキュリティ懸念（認証情報預かり） | 高 | ReadOnlyアクセスのみ、暗号化保存、SOC2準拠をLP明記 |

## 競合分析

| サービス | 月額 | 特徴 | QuotaGuardの優位性 |
|----------|------|------|-------------------|
| AWS Budgets | 無料 | AWS限定、設定が複雑 | マルチクラウド、設定1分 |
| Infracost | 無料〜 | IaCコスト予測（事前） | 実コスト監視（事後） |
| CloudHealth | $$$$ | エンタープライズ向け | 個人開発者価格帯 |
| Vantage | $50〜 | 高機能ダッシュボード | 10分の1の価格 |

## $20達成シナリオ

```
Month 1: 無料ユーザー50人獲得（SEO + Twitter）
Month 2: Pro 2人転換 = $10/月
Month 3: Pro 4人 = $20/月 ✅
Month 4: Pro 6人 + Team 1人 = $42/月
```

**必要有料ユーザー数: Pro 4人で$20達成**

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC（顧客獲得コスト） | $0（SEO/SNS中心） |
| ARPU（ユーザー平均単価） | $5.00 |
| 粗利率 | 96.4%（$5.00 - $0.18/ユーザー） |
| LTV（12ヶ月想定） | $60 |
| LTV/CAC | ∞（オーガニック獲得前提） |
| チャーン率（予測） | 3%/月（保険型→低チャーン） |
| Payback Period | 即時 |
