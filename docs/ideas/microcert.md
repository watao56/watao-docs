# 🏅 MicroCert — デジタル認定証・バッジ作成SaaS

## 概要

オンラインコース修了証・コミュニティバッジ・社内表彰状などのデジタル認定証を簡単に作成・発行・共有できるSaaS。海外ではAccredible ($5M+ ARR)、Credly ($100M+ 買収)、Certifier ($1M+ ARR) が成長中だが、日本語に最適化された低価格サービスは存在しない。

## 海外事例分析

| サービス | 規模 | 特徴 |
|---------|------|------|
| **Credly** (US) | Pearson VUEが$100M+で買収 | エンタープライズ向けデジタルバッジ |
| **Accredible** (US) | $5M+ ARR | 大学・企業向け認定証、ブロックチェーン検証 |
| **Certifier** (PL) | $1M+ ARR | 中小事業者向け、安価、テンプレート豊富 |
| **Sertifier** (TR) | $500K+ ARR | イベント・研修向け |
| **Badgr** (US) | 買収済み | Open Badges標準準拠 |

**日本の現状**: オンライン講座（Udemy日本語10万+コース、ストアカ10万+講師）が急成長中だが、修了証はPDFを手作りか、なし。コミュニティ（Slack/Discord）でのバッジ文化は未発達。

## ターゲット

### プライマリ
- **オンライン講師/コースクリエイター**（Udemy、ストアカ、自前コース）
- 修了証を出したいが作るのが面倒
- 受講生の満足度・完了率を上げたい

### セカンダリ
- **コミュニティ運営者**（Discord/Slackコミュニティ）
- **企業研修担当者**（社内研修の修了証）
- **イベント主催者**（カンファレンス参加証、ハッカソン入賞証）
- **資格スクール・専門学校**

## 料金プラン

| プラン | 月額 | 内容 |
|-------|------|------|
| **Free** | ¥0 | 月5枚発行、テンプレ3種、MicroCertロゴ入り |
| **Starter** | ¥780（~$5.2） | 月50枚、テンプレ15種、ロゴなし、CSV一括発行、検証URL |
| **Pro** | ¥1,580（~$10.5） | 無制限発行、全テンプレ、カスタムデザイン、API、分析、LinkedIn連携 |

### $20達成シナリオ
- **Starter 2人 + Pro 1人** = $10.4 + $10.5 = $20.9/月
- **達成時期**: 2〜3ヶ月目（オンライン講師コミュニティへのリーチ）

## ユーザーフロー

### 発行者（講師/運営者）
```
1. 登録 → テンプレート選択（15種以上）
2. カスタマイズ: ロゴ、色、テキスト、署名
3. 受領者情報入力（名前、メール）or CSV一括アップロード
4. 発行 → 受領者にメール通知
5. ダッシュボードで発行履歴・統計確認
```

### 受領者（受講生）
```
1. メールで認定証リンクを受信
2. 美しいデジタル認定証を表示（microcert.app/verify/[id]）
3. SNSシェアボタン（X、LinkedIn、Facebook）
4. PDF/PNGダウンロード
5. LinkedIn「資格・認定」に追加
```

## デザインコンセプト

### ビジュアルアイデンティティ
- **カラー**: ゴールド × ネイビー × ホワイト（権威と信頼）
- **フォント**: Noto Serif JP（認定証本体）+ Noto Sans JP（UI）
- **テーマ**: 「デジタルでも重みを感じる証明書」

### 認定証テンプレート
- **Classic Certificate**: 伝統的な賞状風（枠飾り、ゴールドアクセント）
- **Modern Minimal**: ミニマル、余白多め、フラットデザイン
- **Tech Badge**: 丸型バッジ、テック企業風
- **Japanese Traditional**: 和風、書道フォント、落款風ロゴ
- **Gradient**: グラデーション背景、コース完了証向け
- **Dark Premium**: ダーク背景、高級感、VIPバッジ向け
- **Playful**: カラフル、イラスト風、子供向け教育
- **Corporate**: 企業研修向け、フォーマル

### 検証ページ
- ユニークURL（microcert.app/verify/[short-id]）
- 発行者情報、発行日、受領者名
- QRコード（名刺やポートフォリオに貼れる）
- OGP画像自動生成（SNSシェア時に認定証画像が表示）

## アーキテクチャ

```
[発行者] → [Next.js Frontend (Vercel)]
                    ↓
            [API Routes]
                    ↓
            [Supabase PostgreSQL + Auth]
                    ↓
            [認定証レンダラー (React → Canvas/SVG)]
                    ↓
            [PDF生成 (jsPDF + html2canvas)]
            [PNG生成 (Sharp)]
            [OGP画像生成 (Satori)]
                    ↓
            [Supabase Storage] → 画像保存
            [Resend] → メール通知
```

### 技術スタック
- **Frontend**: Next.js 14 + Tailwind CSS + Canvas API
- **Backend**: Next.js API Routes
- **DB**: Supabase (PostgreSQL + Auth + Storage)
- **認定証レンダリング**: React → SVG → PNG/PDF (Satori + jsPDF)
- **メール**: Resend（月100通無料、$20/月で50,000通）
- **OGP画像**: Satori (Vercel製、OSS)
- **ホスティング**: Vercel
- **決済**: Stripe

## DB設計

```sql
-- 組織/発行者
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id UUID REFERENCES auth.users(id),
  name TEXT NOT NULL,
  logo_url TEXT,
  plan TEXT DEFAULT 'free',
  certs_this_month INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- テンプレート
CREATE TABLE templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID REFERENCES organizations(id),
  name TEXT NOT NULL,
  design JSONB NOT NULL, -- テンプレート定義（色、フォント、レイアウト等）
  thumbnail_url TEXT,
  is_system BOOLEAN DEFAULT false, -- システム提供テンプレ
  created_at TIMESTAMPTZ DEFAULT now()
);

-- 認定証
CREATE TABLE certificates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  short_id TEXT UNIQUE NOT NULL, -- 検証URL用（8文字ランダム）
  org_id UUID REFERENCES organizations(id),
  template_id UUID REFERENCES templates(id),
  recipient_name TEXT NOT NULL,
  recipient_email TEXT,
  title TEXT NOT NULL, -- 「Webデザイン基礎コース 修了証」等
  description TEXT,
  issued_date DATE DEFAULT CURRENT_DATE,
  expiry_date DATE, -- 有効期限（任意）
  metadata JSONB DEFAULT '{}', -- カスタムフィールド
  image_url TEXT, -- 生成済み画像URL
  ogp_url TEXT,
  status TEXT DEFAULT 'issued', -- draft/issued/revoked
  shared_count INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- 検証ログ
CREATE TABLE verifications (
  id BIGSERIAL PRIMARY KEY,
  certificate_id UUID REFERENCES certificates(id),
  referrer TEXT,
  verified_at TIMESTAMPTZ DEFAULT now()
);

-- サブスクリプション
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID REFERENCES organizations(id),
  stripe_subscription_id TEXT,
  plan TEXT NOT NULL,
  status TEXT DEFAULT 'active',
  current_period_end TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

## コスト見積もり

### インフラコスト（30組織、月300枚発行時）
| 項目 | 月額 |
|------|------|
| Vercel | $0（無料枠） |
| Supabase | $0（無料枠: 500MB DB, 1GB Storage） |
| Resend | $0（月100通無料）→ $20/月（スケール時） |
| ドメイン | $1 |
| **合計** | **~$1/月**（初期）、~$21/月（スケール時） |

### AI費用
- **$0**（AI不使用。テンプレートベースのレンダリング）

## MVPスコープ（2週間）

### Week 1
- [ ] 認証（Supabase Auth）
- [ ] テンプレート4種（Classic/Modern/Tech/Japanese）
- [ ] 認定証カスタマイズUI（テキスト編集、色変更、ロゴアップロード）
- [ ] 認定証レンダリング（SVG → PNG）
- [ ] 検証ページ（/verify/[short-id]）

### Week 2
- [ ] メール通知（Resend）
- [ ] CSV一括発行
- [ ] PDF/PNGダウンロード
- [ ] SNSシェアボタン + OGP画像
- [ ] Stripe決済 + LP

## マーケティング計画

### Phase 1: オンライン講師獲得（1ヶ月目）
- **ストアカ/Udemy講師コミュニティ**に投稿
- X/Twitterで「修了証のビフォーアフター」投稿（手作りPDF vs MicroCert）
- 「オンライン講座 修了証 作り方」のSEO記事
- 人気講師5人に無料Proプラン提供 → 事例作り

### Phase 2: コミュニティ展開（2-3ヶ月目）
- Discord/Slackコミュニティ運営者への提案
- ハッカソン主催者への参加証発行提案
- Product Hunt Launch

### Phase 3: B2B展開（4-6ヶ月目）
- 企業研修向け「チームプラン」
- 専門学校・資格スクールへの営業
- Open Badges 3.0準拠で国際標準対応

## リスク

| リスク | 影響 | 対策 |
|-------|------|------|
| 認定証の価値認識（「デジタルで十分？」） | 中 | LinkedIn連携で実用価値を訴求、検証URLで信頼性担保 |
| 利用頻度の低さ（コース終了時のみ） | 中 | コース単位ではなくモジュール単位のバッジ推奨、コミュニティバッジの常時発行 |
| Canva等の汎用ツール | 低 | 一括発行、検証URL、LinkedIn連携はCanvaにない |
| メール配信コストのスケール | 低 | Resend $20/月で5万通、十分な余裕 |

## 競合分析

| 競合 | 価格 | 弱点 |
|------|------|------|
| Accredible | $150/月〜 | 高額、エンタープライズ向け |
| Certifier | $49/月〜 | 英語のみ、日本の文化に非対応 |
| Canva | 無料〜$12.99/月 | 一括発行不可、検証URL不可 |
| 手作りPDF | 無料 | 時間がかかる、見た目が悪い、検証不可 |

**MicroCertの差別化**: 日本語最適化 × 月$5.2からの低価格（Accredibleの1/30） × テンプレート品質 × 検証URL × SNSシェア最適化

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | ~¥800（コミュニティ+SEO） |
| ARPU（月間） | ¥1,080（Starter/Pro加重平均） |
| 粗利率 | 98.5%（AI不使用） |
| LTV（12ヶ月想定） | ¥12,960 |
| LTV/CAC | 16.2x |
| 月間チャーン | ~6% |
| Payback Period | 1ヶ月未満 |
