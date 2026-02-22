# 🍱 SnapMenu — AI飲食店メニューデジタル化SaaS

## 概要

紙メニューの写真を撮るだけで、多言語対応のデジタルメニュー（QRコード付き）を自動生成するSaaS。インバウンド観光客増加（2025年: 3,600万人超）に対応し、飲食店の言語バリアを解消する。

## 海外事例分析

| サービス | 規模 | 特徴 |
|---------|------|------|
| **Menufy** (US) | $50M+ ARR | オンライン注文統合型デジタルメニュー |
| **GloriaFood** (EU) | $10M+ ARR | 無料デジタルメニュー+注文機能 |
| **Sunday** (FR) | $150M調達 | QRコード決済+メニュー |
| **Mr Yum** (AU) | $89M調達 | ビジュアルメニュー+注文+決済 |

**日本の現状**: 多くの飲食店がまだ紙メニューのみ。英語メニューがあっても品質が低い。Googleレンズで個別翻訳する観光客が多いが体験が悪い。

## ターゲット

### プライマリ
- **個人経営の飲食店**（ラーメン、居酒屋、定食屋）
- 外国人客が来るがメニューが日本語のみ
- ITリテラシー低め、予算もない
- 観光地・駅前・繁華街の店舗

### セカンダリ
- **観光地の土産物店・カフェ**
- **フードトラック・屋台**
- **ホテル内レストラン**

## 料金プラン

| プラン | 月額 | 内容 |
|-------|------|------|
| **Free** | ¥0 | メニュー1ページ、2言語（日英）、SnapMenuロゴ入り |
| **Basic** | ¥500（~$3.3） | メニュー5ページ、5言語、ロゴなし、QRコードPDF |
| **Pro** | ¥980（~$6.5） | 無制限ページ、10言語、写真付きメニュー、アクセス解析、カスタムドメイン |

### $20達成シナリオ
- **Basic 3人 + Pro 2人** = $9.9 + $13.0 = $22.9/月
- **達成時期**: 2〜3ヶ月目（観光地の飲食店にチラシ配布 + Google Maps SEO）

## ユーザーフロー

```
1. LPにアクセス → 無料登録（メール or LINE）
2. 紙メニューをスマホで撮影（1〜5枚）
3. AIが自動認識: 料理名、価格、カテゴリ、アレルゲン
4. 翻訳プレビュー表示（日→英中韓 etc.）
5. デザインテンプレート選択（和風/モダン/カジュアル）
6. QRコード生成 → 印刷用PDF or ステッカー注文
7. 客がQRスキャン → スマホでデジタルメニュー閲覧
```

## デザインコンセプト

### ビジュアルアイデンティティ
- **カラー**: 赤（食欲）× 白 × 黒の和モダン
- **フォント**: Noto Sans JP + 游ゴシック
- **テーマ**: 「紙のぬくもり × デジタルの便利さ」

### メニューテンプレート
- **和風**: 縦書き風、筆書きフォント、和紙テクスチャ
- **モダン**: ミニマル、写真大きめ、Instagram風
- **カジュアル**: ポップ、アイコン多用、ファミレス風
- **高級**: 黒背景、金文字、コース料理向け

### 管理画面
- スマホファースト（店主がスマホで完結）
- 写真撮影→即反映のリアルタイムプレビュー
- ドラッグ&ドロップでメニュー順序変更

## アーキテクチャ

```
[スマホカメラ] → [Next.js Frontend (Vercel)]
                      ↓
              [API Routes / Edge Functions]
                      ↓
              [Google Cloud Vision API] → OCR
                      ↓
              [GPT-4o-mini] → 構造化 + 翻訳
                      ↓
              [Supabase PostgreSQL] → 保存
                      ↓
              [QRコード生成 (qrcode.js)]
                      ↓
              [Vercel Edge] → メニュー配信（CDN）
```

### 技術スタック
- **Frontend**: Next.js 14 (App Router) + Tailwind CSS + shadcn/ui
- **Backend**: Next.js API Routes (Vercel Serverless)
- **DB**: Supabase (PostgreSQL + Auth + Storage)
- **OCR**: Google Cloud Vision API（月1000リクエスト無料）
- **AI翻訳・構造化**: GPT-4o-mini
- **ホスティング**: Vercel (無料枠)
- **QRコード**: qrcode.js (OSS)
- **画像ストレージ**: Supabase Storage (1GB無料)

## DB設計

```sql
-- 店舗
CREATE TABLE shops (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id UUID REFERENCES auth.users(id),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  plan TEXT DEFAULT 'free', -- free/basic/pro
  logo_url TEXT,
  theme TEXT DEFAULT 'modern',
  languages TEXT[] DEFAULT '{ja,en}',
  created_at TIMESTAMPTZ DEFAULT now()
);

-- メニューページ
CREATE TABLE menu_pages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  shop_id UUID REFERENCES shops(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  sort_order INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- メニューアイテム
CREATE TABLE menu_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  page_id UUID REFERENCES menu_pages(id) ON DELETE CASCADE,
  name_ja TEXT NOT NULL,
  translations JSONB DEFAULT '{}', -- {"en": "Ramen", "zh": "拉面", ...}
  price INT, -- 円
  description_ja TEXT,
  desc_translations JSONB DEFAULT '{}',
  photo_url TEXT,
  category TEXT,
  allergens TEXT[],
  is_vegetarian BOOLEAN DEFAULT false,
  is_halal BOOLEAN DEFAULT false,
  sort_order INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- アクセスログ（Pro用）
CREATE TABLE menu_views (
  id BIGSERIAL PRIMARY KEY,
  shop_id UUID REFERENCES shops(id),
  language TEXT,
  user_agent TEXT,
  viewed_at TIMESTAMPTZ DEFAULT now()
);

-- サブスクリプション
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  shop_id UUID REFERENCES shops(id),
  stripe_subscription_id TEXT,
  plan TEXT NOT NULL,
  status TEXT DEFAULT 'active',
  current_period_end TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

## コスト見積もり

### インフラコスト（50ユーザー時）
| 項目 | 月額 |
|------|------|
| Vercel | $0（無料枠） |
| Supabase | $0（無料枠: 500MB DB, 1GB Storage） |
| Google Cloud Vision | $0（月1000リクエスト無料、超過分$1.50/1000） |
| GPT-4o-mini | ~$2（メニュー構造化+翻訳、1店舗あたり初回$0.05程度） |
| ドメイン | $1（年$12） |
| **合計** | **~$3/月** |

### AI費用詳細
- メニュー1ページのOCR+構造化+5言語翻訳: ~$0.01（GPT-4o-mini）
- 初回セットアップ時のみ発生、更新時は差分のみ
- 月間新規10店舗 × 平均3ページ = $0.30

## MVPスコープ（2週間）

### Week 1
- [ ] 認証（Supabase Auth: メール）
- [ ] メニュー写真アップロード → OCR → 構造化
- [ ] 日英翻訳の自動生成
- [ ] メニュー表示ページ（/shop/[slug]）
- [ ] QRコード生成

### Week 2
- [ ] テンプレート3種（和風/モダン/カジュアル）
- [ ] メニュー編集UI（アイテム追加・削除・並替）
- [ ] Stripe決済（Basic/Pro）
- [ ] QRコードPDFダウンロード
- [ ] LP + 利用規約

## マーケティング計画

### Phase 1: 種まき（1ヶ月目）
- 観光地（浅草、京都、大阪道頓堀）の飲食店に**チラシ配布**（QRで無料体験）
- Google Maps上の飲食店に**DMメッセージ**（英語メニューなしの店を狙う）
- X/Twitterで「外国人が困る日本のメニュー」ネタでバズ狙い
- インバウンド系メディア（MATCHA、Japan Guide）に紹介依頼

### Phase 2: 拡大（2-3ヶ月目）
- **成功事例動画**: 「この居酒屋、外国人客が3倍になりました」
- 商工会議所・観光協会との連携
- 飲食店オーナー向けFacebookグループでの口コミ
- Product Hunt Launch（英語圏のレストランオーナーも獲得）

### Phase 3: スケール（4-6ヶ月目）
- ホテル・旅館チェーンへのB2B営業
- 多言語メニューコンサルとしてのアップセル
- アレルゲン表示義務化を追い風にした訴求

## リスク

| リスク | 影響 | 対策 |
|-------|------|------|
| OCR精度（手書きメニュー） | 中 | 手動補正UIを用意、フィードバックで改善 |
| 飲食店のITリテラシー | 高 | LINE連携、電話サポート、動画チュートリアル |
| メニュー変更の頻度 | 低 | リアルタイム編集UI、写真再撮影で上書き |
| 競合（Googleレンズ等） | 中 | 「店舗ブランディング」と「常設」で差別化 |
| インバウンド減少リスク | 中 | 在日外国人（300万人）もターゲットに |

## 競合分析

| 競合 | 価格 | 弱点 |
|------|------|------|
| Googleレンズ | 無料 | その場限り、店のブランド感なし、精度不安定 |
| Wovn.io | 高額（月数万円） | 大企業向け、飲食店には高すぎる |
| 手動翻訳 | 1回数万円 | 更新のたびに費用、時間がかかる |
| UberEats等 | 手数料35% | 注文プラットフォームへの依存 |

**SnapMenuの差別化**: 写真1枚で完了する圧倒的手軽さ × 月500円の低価格 × 美しいデザインテンプレート

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC（顧客獲得コスト） | ~¥500（チラシ印刷+配布） |
| ARPU（月間） | ¥740（Basic/Pro加重平均） |
| 粗利率 | 95.2% |
| LTV（12ヶ月想定） | ¥8,880 |
| LTV/CAC | 17.8x |
| 月間チャーン | ~5%（季節変動あり） |
| Payback Period | 1ヶ月未満 |
