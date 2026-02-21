# 🏆 TestiWall — テスティモニアル収集＆ソーシャルプルーフWidget

## 概要

Webサイトやランディングページに貼れる「お客様の声」収集＆表示ウィジェットSaaS。フォームURL1つで顧客からテキスト/動画テスティモニアルを収集し、美しいウォールやカルーセルとして埋め込みコード1行で表示。日本語フォント・縦書き・和風デザインに完全対応。

## 海外事例分析

| サービス | MRR | 特徴 |
|---------|-----|------|
| **Senja** | $50K+ MRR | テスティモニアル収集＆ウィジェット。Indie Hacker発、2年で急成長 |
| **Testimonial.to** | $30K+ MRR | 動画テスティモニアル特化。Product Hunt #1 |
| **Shoutout** | $10K+ MRR | ソーシャルプルーフウィジェット |
| **TrustPulse** | $20K+ MRR | リアルタイム購入通知ポップアップ |

**日本市場の状況**: 日本語対応のテスティモニアルツールはほぼ皆無。企業は手動でスクショを貼るか、WordPressプラグイン（英語UI）を使うのみ。フリーランス・中小事業者の「お客様の声」管理は完全にブルーオーシャン。

## ターゲット

### プライマリ
- **フリーランス・個人事業主**: ポートフォリオやLP用にお客様の声を集めたい
- **中小ECサイトオーナー**: レビューをサイトに美しく表示したい
- **SaaS/Webサービス運営者**: ソーシャルプルーフで転換率を上げたい

### セカンダリ
- コーチ・講師・コンサル業
- 結婚式場、レストラン、美容院

## 料金

| プラン | 月額 | 内容 |
|-------|------|------|
| **Free** | ¥0 | テスティモニアル10件、ウィジェット1個、TestiWallロゴ表示 |
| **Pro** | ¥980/月 | 無制限、ウィジェット5個、ロゴ非表示、動画対応、カスタムデザイン |
| **Business** | ¥2,980/月 | Pro全機能＋チーム、API、優先サポート |

## ユーザーフロー

```
1. サインアップ（Google/GitHub OAuth）
2. プロジェクト作成（サイト名入力）
3. 収集フォームURL自動生成
4. URLを顧客にメール/SNSで送付
5. 顧客がフォームからテスティモニアル投稿（テキスト/動画/星評価）
6. ダッシュボードで承認/編集
7. ウィジェットデザイン選択（ウォール/カルーセル/バッジ/マスonry）
8. 埋め込みコード1行をサイトにコピペ
9. サイト上にリアルタイム表示
```

## デザインコンセプト

- **"信頼を美しく見せる"**: 和紙テクスチャ風背景、日本語Webフォント対応
- ウィジェットテンプレート8種（Masonry Wall / カルーセル / フローティングバッジ / ミニマル引用 / カード / グリッド / スライダー / タイムライン）
- ダークモード完全対応
- アニメーション付きフェードイン表示
- 星評価のカスタムカラー

## アーキテクチャ

```
[収集フォーム] → [API Gateway (Next.js API Routes)]
                         ↓
                   [PostgreSQL (Supabase)]
                         ↓
              [Widget CDN (CloudFront + S3)]
                         ↓
              [埋め込みJS → ユーザーサイト表示]
```

### コンポーネント
- **フロントエンド**: Next.js 14 (App Router) + Tailwind CSS
- **バックエンド**: Next.js API Routes (Serverless)
- **DB**: Supabase (PostgreSQL + Auth + Storage)
- **ウィジェット配信**: CloudFront CDN
- **動画**: Supabase Storage（Free tier 1GB）
- **決済**: Stripe
- **ホスティング**: Vercel (Hobby → Pro)

## DB設計

```sql
-- ユーザー
users (
  id UUID PK,
  email TEXT UNIQUE,
  name TEXT,
  plan TEXT DEFAULT 'free',
  stripe_customer_id TEXT,
  created_at TIMESTAMPTZ
)

-- プロジェクト
projects (
  id UUID PK,
  user_id UUID FK → users,
  name TEXT,
  slug TEXT UNIQUE, -- 収集フォームURL用
  settings JSONB, -- フォームカスタマイズ
  created_at TIMESTAMPTZ
)

-- テスティモニアル
testimonials (
  id UUID PK,
  project_id UUID FK → projects,
  author_name TEXT,
  author_title TEXT,
  author_avatar_url TEXT,
  content TEXT,
  rating INT, -- 1-5
  video_url TEXT,
  status TEXT DEFAULT 'pending', -- pending/approved/archived
  source TEXT, -- form/import/manual
  created_at TIMESTAMPTZ
)

-- ウィジェット
widgets (
  id UUID PK,
  project_id UUID FK → projects,
  type TEXT, -- wall/carousel/badge/etc
  config JSONB, -- デザイン設定
  testimonial_ids UUID[], -- 表示するテスティモニアル
  created_at TIMESTAMPTZ
)
```

## コスト見積もり

| 項目 | 月額 |
|------|------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| CloudFront (CDN) | ~$0.50 |
| ドメイン | ~$1 |
| **合計** | **~$1.50/月** |

※AI不使用。外部API費$0。

## MVPスコープ（2週間）

### Week 1
- [ ] Google Authログイン
- [ ] プロジェクトCRUD
- [ ] 収集フォーム（テキスト+星評価）
- [ ] テスティモニアル一覧＆承認

### Week 2
- [ ] ウィジェット生成（Masonry Wall + カルーセル）
- [ ] 埋め込みコード生成
- [ ] Stripe決済連携
- [ ] LPページ

## マーケ計画

### フェーズ1（ローンチ〜1ヶ月）
- Product Hunt日本語コミュニティ投稿
- Twitter/X で「お客様の声を5分で美しく表示」デモ動画
- Zenn/note にフリーランス向け記事
- indie hacker系コミュニティ

### フェーズ2（2-3ヶ月）
- SEO: 「テスティモニアル 集め方」「お客様の声 表示」
- フリーランスコミュニティ（Lancers, CrowdWorks）でPR
- 無料プランユーザーからのPro転換

### フェーズ3（3-6ヶ月）
- WordPress/Shopifyプラグイン化
- パートナーシップ（LP制作会社）

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フレームワーク | Next.js 14 (App Router) |
| スタイリング | Tailwind CSS + shadcn/ui |
| DB/Auth/Storage | Supabase |
| CDN | CloudFront |
| 決済 | Stripe |
| デプロイ | Vercel |
| ウィジェット | Vanilla JS (軽量、フレームワーク非依存) |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| ウィジェットの表示速度 | UX低下 | CDN配信、JS最小化（<10KB gzip） |
| スパム投稿 | フォーム悪用 | reCAPTCHA、承認制 |
| 競合参入 | 価格競争 | 日本語特化・和風デザインで差別化 |
| 無料プランだけ使われる | 収益化困難 | Free制限を適切に（10件/ロゴ表示） |

## 競合分析

| 競合 | 強み | 弱み | TestiWallの優位性 |
|------|------|------|------------------|
| Senja | 機能豊富、英語圏で人気 | 日本語UI/フォントなし、$29/月〜 | 日本語完全対応、¥980/月 |
| Google Forms+手動 | 無料 | デザイン性ゼロ、手動作業 | ワンクリック美麗表示 |
| WordPress ReviewプラグIn | WPエコシステム | WP限定、英語UI | フレームワーク不問 |

## $20達成シナリオ

```
目標: $20/月 ≒ ¥3,000/月

シナリオ: Pro 3人 + Free多数
- Pro ¥980 × 3 = ¥2,940 ≒ $20
- 必要Free登録: ~50人（Pro転換率6%）
- 達成時期: ローンチ後2-3ヶ月

楽観シナリオ: 
- Pro 5人 × ¥980 = ¥4,900 ≒ $33
- ローンチ後1-2ヶ月
```

## ユニットエコノミクス

```
ARPU（Pro）: ¥980/月
インフラ限界費用/ユーザー: ~¥2/月（CDN転送量）
粗利: ¥978/月 (99.8%)
Stripe手数料: 3.6% = ¥35
純利: ¥943/ユーザー/月

LTV（想定12ヶ月）: ¥11,316
CAC目標: <¥2,000（オーガニック中心なら¥0）
LTV/CAC: >5x ✅

損益分岐: Pro 1人で黒字（インフラ~$1.50/月）
```
