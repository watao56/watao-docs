# 🎨 PromptStock — AIプロンプトマーケットプレイスSaaS

## 概要

高品質なAIプロンプト（Midjourney、DALL-E、Stable Diffusion、ChatGPT等）を売買できる日本語特化マーケットプレイス。海外ではPromptBase ($2M+ 年間売上) が成長中だが、日本語プロンプト・日本文化に特化したマーケットプレイスは存在しない。クリエイターが作ったプロンプトを販売し、プラットフォーム手数料で収益化。

## 海外事例分析

| サービス | 規模 | 特徴 |
|---------|------|------|
| **PromptBase** (US) | $2M+/年売上 | マーケットプレイス、手数料20% |
| **PromptHero** (US) | 月間500万PV | 無料プロンプト共有+Pro$9/月 |
| **AIPRM** (US) | $5M+ ARR | ChatGPTプロンプト拡張機能 |
| **FlowGPT** (US) | $6M調達 | プロンプト共有SNS |
| **ChatX** (US) | $3M+/年 | プロンプトテンプレートSaaS |

**日本の現状**: 日本語AIプロンプトの需要は急増（Midjourney日本語ユーザー50万人超）だが、質の高いプロンプトを見つけるのが困難。noteやBrainで断片的に販売されているが、検索性・品質保証・プレビューがない。

## ターゲット

### バイヤー（購入者）
- **AIイラストレーター**: 商用利用可能な高品質プロンプトが欲しい
- **デザイナー/マーケター**: 広告素材・バナーのAI生成プロンプトが欲しい
- **コンテンツクリエイター**: YouTube/ブログ用の画像生成プロンプト
- **ビジネスパーソン**: ChatGPT業務効率化プロンプト

### セラー（販売者）
- **プロンプトエンジニア**: 技術を収益化したい
- **AIアーティスト**: 作品と共にプロンプトを販売
- **業務効率化のプロ**: ChatGPTプロンプトテンプレートを販売

## 料金モデル

### マーケットプレイス手数料
| 項目 | 内容 |
|------|------|
| プロンプト販売価格 | セラーが設定（¥100〜¥5,000） |
| プラットフォーム手数料 | **30%**（業界標準） |
| セラー取り分 | **70%** |
| 最低販売価格 | ¥100 |

### サブスクリプション（バイヤー向け）
| プラン | 月額 | 内容 |
|-------|------|------|
| **Free** | ¥0 | 無料プロンプト閲覧、月3つまでDL |
| **Plus** | ¥580（~$3.8） | 月20プロンプトDL、プレミアムプロンプト10%OFF |
| **Unlimited** | ¥1,480（~$9.8） | 無制限DL、全プレミアム含む、新作先行アクセス |

### $20達成シナリオ
- **手数料収入**: 月間プロンプト販売¥20,000（20件×¥1,000平均）→ 手数料¥6,000（~$40）
- **Plus 3人 + Unlimited 1人** = $11.4 + $9.8 = $21.2
- **複合**: 手数料$10 + サブスク$12 = $22/月
- **達成時期**: 2〜3ヶ月目

## ユーザーフロー

### バイヤー
```
1. LP → カテゴリ/AIモデル/スタイルで検索
2. プロンプト詳細ページ → プレビュー画像4枚 + 説明
3. 購入（Stripe/PayPay）→ プロンプトテキスト表示
4. コピー → Midjourney等で使用
5. レビュー投稿
```

### セラー
```
1. 登録 → セラーダッシュボード
2. プロンプト登録: テキスト + プレビュー画像4枚 + カテゴリ + 対応AIモデル
3. 価格設定 → 公開
4. 売上ダッシュボードで収益確認
5. 月末に銀行振込/PayPay送金
```

## デザインコンセプト

### ビジュアルアイデンティティ
- **カラー**: ディープパープル × ネオンピンク × ダークBG（AIアート感）
- **フォント**: Inter + Noto Sans JP
- **テーマ**: 「AIアートのPinterest」— ビジュアルファーストのギャラリー体験

### UI特徴
- **Pinterestスタイルのグリッド**: プレビュー画像がメイン、テキストは最小限
- **ホバーでプロンプト概要表示**: 購入前に雰囲気がわかる
- **AIモデル別フィルター**: Midjourney / DALL-E 3 / Stable Diffusion / Flux / ChatGPT
- **スタイル別タグ**: アニメ / フォトリアル / 水彩 / サイバーパンク / 和風 / ミニマル
- **トレンドセクション**: 今週の人気プロンプトTop10

## アーキテクチャ

```
[ユーザー] → [Next.js Frontend (Vercel)]
                    ↓
            [API Routes]
                    ↓
            [Supabase PostgreSQL + Auth + Storage]
                    ↓
            [Stripe] → 決済
            [Supabase Storage] → プレビュー画像
```

### 技術スタック
- **Frontend**: Next.js 14 + Tailwind CSS + Masonry Grid
- **Backend**: Next.js API Routes
- **DB**: Supabase (PostgreSQL + Auth + Storage)
- **決済**: Stripe (Connect for marketplace payouts)
- **画像ストレージ**: Supabase Storage + Vercel Image Optimization
- **検索**: Supabase Full-Text Search (pg_trgm)
- **ホスティング**: Vercel

## DB設計

```sql
-- ユーザー
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  display_name TEXT NOT NULL,
  avatar_url TEXT,
  bio TEXT,
  role TEXT DEFAULT 'buyer', -- buyer/seller/both
  stripe_account_id TEXT, -- Stripe Connect for sellers
  plan TEXT DEFAULT 'free',
  created_at TIMESTAMPTZ DEFAULT now()
);

-- プロンプト
CREATE TABLE prompts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  seller_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  prompt_text TEXT NOT NULL, -- 購入後のみ表示
  ai_model TEXT NOT NULL, -- midjourney/dalle3/sdxl/flux/chatgpt
  category TEXT NOT NULL, -- illustration/photo/logo/business/writing
  tags TEXT[],
  price INT NOT NULL, -- 円
  preview_urls TEXT[], -- プレビュー画像URL（4枚）
  sales_count INT DEFAULT 0,
  rating_avg DECIMAL(3,2) DEFAULT 0,
  rating_count INT DEFAULT 0,
  status TEXT DEFAULT 'published',
  created_at TIMESTAMPTZ DEFAULT now()
);

-- 購入履歴
CREATE TABLE purchases (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  buyer_id UUID REFERENCES profiles(id),
  prompt_id UUID REFERENCES prompts(id),
  price_paid INT NOT NULL,
  platform_fee INT NOT NULL, -- 30%
  seller_payout INT NOT NULL, -- 70%
  stripe_payment_id TEXT,
  purchased_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(buyer_id, prompt_id)
);

-- レビュー
CREATE TABLE reviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  purchase_id UUID REFERENCES purchases(id),
  buyer_id UUID REFERENCES profiles(id),
  prompt_id UUID REFERENCES prompts(id),
  rating INT CHECK (rating BETWEEN 1 AND 5),
  comment TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- サブスクリプション
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id),
  stripe_subscription_id TEXT,
  plan TEXT NOT NULL,
  status TEXT DEFAULT 'active',
  downloads_this_month INT DEFAULT 0,
  current_period_end TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

## コスト見積もり

### インフラコスト（100ユーザー時）
| 項目 | 月額 |
|------|------|
| Vercel | $0（無料枠） |
| Supabase | $0（無料枠: 500MB DB, 1GB Storage） |
| Stripe手数料 | 3.6%（決済額に対して） |
| ドメイン | $1 |
| **合計** | **~$2/月** + Stripe手数料 |

### AI費用
- **$0**（AI不使用。マーケットプレイスのみ）

## MVPスコープ（2週間）

### Week 1
- [ ] 認証（Supabase Auth）
- [ ] プロンプト登録・表示（CRUD）
- [ ] Pinterestスタイルのギャラリーページ
- [ ] カテゴリ・AIモデルフィルター
- [ ] 検索機能（pg_trgm）

### Week 2
- [ ] Stripe決済（単品購入）
- [ ] 購入後のプロンプトテキスト表示
- [ ] セラーダッシュボード（売上表示）
- [ ] レビュー機能
- [ ] LP + SEO

### Week 3（追加）
- [ ] サブスクリプション（Plus/Unlimited）
- [ ] Stripe Connect（セラーへの送金）
- [ ] トレンドセクション

## マーケティング計画

### Phase 1: コンテンツシーディング（1ヶ月目）
- **自分で30〜50個の高品質プロンプトを投稿**（初期在庫）
- X/Twitterで「このイラスト、プロンプト1つで生成しました」系の投稿
- AIイラストコミュニティ（Pixiv、X）でのプロモーション
- 「Midjourney プロンプト 日本語」のSEO対策

### Phase 2: セラー獲得（2-3ヶ月目）
- AIアーティスト10人にDMでセラー招待（初月手数料0%キャンペーン）
- noteでプロンプト販売していた人をリクルート
- プロンプトコンテスト開催（賞金¥10,000）

### Phase 3: コミュニティ化（4-6ヶ月目）
- セラーランキング、フォロー機能
- コレクション/キュレーション機能
- API提供（プロンプトライブラリを自分のアプリに組み込み）

## リスク

| リスク | 影響 | 対策 |
|-------|------|------|
| 鶏と卵問題（セラー/バイヤー） | 高 | 初期は自分でプロンプト投稿、セラー手数料0%キャンペーン |
| プロンプト品質の担保 | 中 | プレビュー画像必須、レビュー機能、品質基準ガイドライン |
| AIモデルの進化で陳腐化 | 中 | 新モデル対応を迅速に、ChatGPTプロンプトは長寿命 |
| 著作権・ライセンス問題 | 中 | 利用規約で商用利用条件を明記、セラーに著作権保証を求める |
| PromptBaseの日本進出 | 低 | 日本語UI/UX、PayPay対応、日本文化プロンプトで差別化 |

## 競合分析

| 競合 | 価格 | 弱点 |
|------|------|------|
| PromptBase | $1.99〜 | 英語のみ、日本語プロンプト少ない |
| note | 自由 | 検索性悪い、プレビューなし、プロンプト特化でない |
| Brain | 自由 | 情報商材感が強い、AI特化でない |
| Pixiv FANBOX | ¥100〜 | サブスク型、プロンプト単品販売に不向き |

**PromptStockの差別化**: 日本語プロンプト特化 × Pinterestスタイルのビジュアルギャラリー × プレビュー画像必須の品質保証 × PayPay対応

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC（バイヤー） | ~¥300（SNS+SEO） |
| CAC（セラー） | ~¥0（DM招待） |
| ARPU（月間、手数料+サブスク） | ¥800 |
| 粗利率 | 97.5%（AI不使用） |
| LTV（12ヶ月想定） | ¥9,600 |
| LTV/CAC | 32x |
| 月間チャーン | ~8% |
| Payback Period | 1ヶ月未満 |
