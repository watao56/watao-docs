# 📸 PixelProof — デプロイ後ビジュアルリグレッション自動検知

## 概要

WebサイトのデプロイをWebhookで検知し、自動でスクリーンショットを撮影。前回デプロイ時と画像比較（ピクセル差分）して、UIの意図しない崩れ・レイアウトずれを検知＆Slack/メールで通知。「CSSを1行変えたら別ページが崩れてた」を自動で発見。

## ターゲット

- **メイン**: フリーランスWeb制作者・小規模Web制作会社（5-20サイト運用）
- **サブ**: 個人開発者（自分のSaaSのUI品質管理）
- **ペルソナ**: 鈴木さん（35歳）Web制作フリーランス。10社のWordPressサイトを保守。先月、プラグイン更新後に1つのサイトのお問い合わせフォームがレイアウト崩れ。クライアントから「フォームが使えない」と連絡が来るまで5日間気づかなかった。

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 1サイト、5ページ、週1回チェック |
| Pro | $7/月 | 10サイト、各50ページ、デプロイ時+日次チェック、Slack通知 |
| Agency | $18/月 | 50サイト、無制限ページ、優先チェック、クライアント向けレポート |

## ユーザーフロー

1. サインアップ（メール or GitHub OAuth）
2. 監視URLを登録（トップ、お問い合わせ、料金ページ等）
3. ベースラインスクリーンショット自動撮影
4. Webhook URL発行（Vercel/Netlify/GitHub Actionsに設定）or 日次自動チェック
5. デプロイ検知 → 全ページ自動スクリーンショット → ベースラインと比較
6. 差分がしきい値（デフォルト: 0.5%）を超えたら通知
7. ダッシュボードで差分ビジュアル確認（Before/After/Diff）
8. 「問題なし」なら新しいベースラインとして承認

## アーキテクチャ

```
[Vercel/Netlify Webhook] or [EventBridge 日次]
        ↓
[AWS Lambda (オーケストレーター)]
        ↓
[AWS Lambda + Puppeteer Layer (スクリーンショット撮影)]
        ↓
[S3 (スクリーンショット保存)]
        ↓
[Lambda (pixelmatch で画像比較)]
        ↓ (差分 > しきい値)
[SNS → Slack/Email 通知 + 差分画像添付]

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

CREATE TABLE sites (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  name TEXT NOT NULL,
  base_url TEXT NOT NULL,
  webhook_secret TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE pages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  site_id UUID REFERENCES sites(id),
  path TEXT NOT NULL, -- '/', '/contact', '/pricing'
  viewport TEXT DEFAULT '1280x720',
  threshold DECIMAL DEFAULT 0.5, -- 差分しきい値 %
  baseline_screenshot_key TEXT, -- S3 key
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE snapshots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  page_id UUID REFERENCES pages(id),
  screenshot_key TEXT NOT NULL, -- S3 key
  diff_key TEXT, -- S3 key (差分画像)
  diff_percent DECIMAL,
  status TEXT DEFAULT 'pending', -- pending, pass, fail, approved
  triggered_by TEXT, -- webhook, schedule
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE notification_channels (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  type TEXT NOT NULL,
  config JSONB NOT NULL,
  enabled BOOLEAN DEFAULT true
);
```

## コスト見積もり

| 項目 | 月額 |
|------|------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| AWS Lambda + Puppeteer | $1.50 (100サイト×50ページ×日次) |
| S3 (スクリーンショット保存) | $0.50 (10GB) |
| **合計** | **$2.00/月** (100ユーザー時) |

## MVPスコープ（2週間）

### Week 1
- 認証（Supabase Auth）
- サイト＆ページ登録UI
- Lambda + Puppeteer: スクリーンショット撮影
- S3保存 + pixelmatch差分比較

### Week 2
- Webhook受信エンドポイント（Vercel/Netlify対応）
- Slack/メール通知（差分画像付き）
- Before/After/Diffビューワー
- LP + Stripe決済

## マーケ計画

1. **SEO**: 「CSS 崩れ 検知」「デプロイ後 レイアウト確認」で記事
2. **Twitter/X**: Web制作者コミュニティに刺さるデモ動画
3. **WordPress界隈**: 「プラグイン更新後のレイアウト崩れ自動検知」として訴求
4. **Zenn/Qiita**: 「ビジュアルリグレッションテストを$7で自動化する方法」
5. **Product Hunt**: 開発者ツールとしてローンチ
6. **Web制作系Slack/Discord**: コミュニティで紹介

## 技術スタック

- **フロントエンド**: Next.js 14 + Tailwind CSS + shadcn/ui
- **バックエンド**: AWS Lambda (Node.js) + Puppeteer (chrome-aws-lambda)
- **DB**: Supabase (PostgreSQL)
- **ストレージ**: AWS S3
- **画像比較**: pixelmatch (npm)
- **認証**: Supabase Auth
- **決済**: Stripe

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| 動的コンテンツによる偽陽性（広告、タイムスタンプ） | 高 | 除外エリア設定機能、しきい値調整 |
| Puppeteer Lambda Layer のサイズ制限 | 中 | chrome-aws-lambdaは実績あり。Fargate Spotへのフォールバック |
| S3ストレージコスト増大 | 低 | 古いスナップショットの自動削除（30日保持） |
| Percy/Chromatic等の競合 | 中 | 価格差10倍以上。ノーコード設定で差別化 |

## 競合分析

| サービス | 月額 | 特徴 | PixelProofの優位性 |
|----------|------|------|-------------------|
| Percy (BrowserStack) | $99〜 | CI/CDパイプライン統合 | 価格1/14、ノーコード設定 |
| Chromatic | $149〜 | Storybook特化 | Storybookなしで使える |
| Diffy | $29〜 | ビジュアルテスト | 価格1/4、Webhook対応 |
| 手動チェック | $0 | 全ページ目視確認 | 自動化、見落としゼロ |

## $20達成シナリオ

```
Month 1: 無料ユーザー30人（Web制作コミュニティ）
Month 2: Pro 2人 = $14/月
Month 3: Pro 3人 = $21/月 ✅
Month 4: Pro 4人 + Agency 1人 = $46/月
```

**必要有料ユーザー数: Pro 3人で$21達成**

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0 |
| ARPU | $7.00 |
| 粗利率 | 97.1%（$7.00 - $0.02/ユーザー） |
| LTV（12ヶ月） | $84 |
| LTV/CAC | ∞ |
| チャーン率（予測） | 3%/月（保守契約に紐づく→低チャーン） |
| Payback Period | 即時 |
