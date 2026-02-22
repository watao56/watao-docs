# 📇 DigiMeishi — デジタル名刺SaaS

## 概要

スマホで完結するデジタル名刺サービス。QRコード/NFCで瞬時に連絡先交換ができ、交換履歴・フォローアップリマインダー付き。海外ではPopl ($10M+ ARR)、HiHello ($5M+ ARR)、Blinq ($12M調達) が成長中だが、日本の「名刺文化」に最適化されたサービスは皆無。

## 海外事例分析

| サービス | 規模 | 特徴 |
|---------|------|------|
| **Popl** (US) | $10M+ ARR | NFCカード/ステッカー + デジタルプロフィール |
| **HiHello** (US) | $5M+ ARR | デジタル名刺 + 名刺スキャン + CRM連携 |
| **Blinq** (AU) | $12M調達 | Apple Wallet / Google Wallet連携 |
| **Linq** (US) | $8M調達 | NFC製品 + SaaS + エンタープライズ |
| **Dot** (KR) | $10M+ | NFCカード+アプリ、韓国で急成長 |

**日本の現状**: 名刺交換は年間100億枚（推定）。リモートワーク増加でオンライン名刺交換の需要増。Sansan ($1B+ 上場) は法人向け名刺管理だが高額。個人/フリーランス向けの「おしゃれなデジタル名刺」は空白。

## ターゲット

### プライマリ
- **フリーランス・個人事業主**（デザイナー、エンジニア、コンサル）
- 名刺をおしゃれにしたい、紙を持ち歩きたくない
- 交流会・カンファレンスに頻繁に参加

### セカンダリ
- **スタートアップ社員**（名刺発注が面倒、肩書きが変わりやすい）
- **副業ワーカー**（本業と副業で名刺を分けたい）
- **学生**（就活・インターン用）
- **営業職**（大量の名刺交換 + フォローアップ管理）

## 料金プラン

| プラン | 月額 | 内容 |
|-------|------|------|
| **Free** | ¥0 | 名刺1枚、基本デザイン5種、QRコード、月10回交換 |
| **Plus** | ¥480（~$3.2） | 名刺3枚（本業/副業/プライベート）、全デザイン、無制限交換、交換履歴、vCard出力 |
| **Pro** | ¥980（~$6.5） | 無制限名刺、カスタムデザイン、フォローアップリマインダー、分析、Apple Wallet連携、NFCカード注文 |

### $20達成シナリオ
- **Plus 4人 + Pro 1人** = $12.8 + $6.5 = $19.3/月 → 追加1人で超過
- **Pro 3人 + Plus 1人** = $19.5 + $3.2 = $22.7/月
- **達成時期**: 2ヶ月目（フリーランスコミュニティでの口コミ）

## ユーザーフロー

```
1. LPにアクセス → Google/Apple/メール登録
2. プロフィール入力（名前、肩書き、連絡先、SNSリンク）
3. デザインテンプレート選択 → リアルタイムプレビュー
4. 名刺ページ公開（digimeishi.app/your-name）
5. QRコード生成 → スマホのウィジェット/ロック画面に設置
6. 対面: QRを見せる → 相手がスキャン → 連絡先を一発保存
7. 交換履歴に自動記録 → フォローアップリマインダー設定
```

## デザインコンセプト

### ビジュアルアイデンティティ
- **カラー**: ブラック × ゴールド（高級感）+ ホワイトモード切替
- **フォント**: Noto Sans JP + Outfit（モダン英字）
- **テーマ**: 「紙の名刺を超える、デジタルの品格」

### 名刺デザインテンプレート
- **Classic**: 伝統的な横型名刺レイアウト、品格あり
- **Minimal**: 余白多め、1色+モノクロ、デザイナー向け
- **Gradient**: グラデーション背景、テック系・スタートアップ向け
- **Photo**: 写真背景、クリエイター・インフルエンサー向け
- **Dark**: ダーク背景、エンジニア・テック向け
- **Japanese**: 和柄、筆書きフォント、伝統的ビジネス向け
- **Neon**: ネオンカラー、イベント・DJ・エンタメ向け

### プロフィールページ
- カード風のアニメーション（ホバーで3D回転）
- SNSリンクはアイコンボタン（X、Instagram、GitHub、LinkedIn等）
- 「連絡先を保存」ボタン → vCardダウンロード
- 自己紹介テキスト + ポートフォリオリンク

## アーキテクチャ

```
[スマホ/PC] → [Next.js Frontend (Vercel)]
                    ↓
            [API Routes]
                    ↓
            [Supabase PostgreSQL + Auth]
                    ↓
            [プロフィールページ (SSG/ISR)]
                    ↓
            [QRコード生成 (qrcode.js)]
            [vCard生成 (vcf.js)]
            [Apple Wallet Pass (passkit-generator)]
```

### 技術スタック
- **Frontend**: Next.js 14 + Tailwind CSS + Framer Motion
- **Backend**: Next.js API Routes
- **DB**: Supabase (PostgreSQL + Auth + Realtime)
- **ホスティング**: Vercel (ISR でプロフィールページを高速配信)
- **QRコード**: qrcode.js
- **vCard**: vcf.js (OSS)
- **Apple Wallet**: passkit-generator (OSS)
- **決済**: Stripe
- **NFCカード**: 外部印刷業者連携（Alibaba等でNFCカード$1/枚）

## DB設計

```sql
-- ユーザー
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  display_name TEXT NOT NULL,
  email TEXT,
  plan TEXT DEFAULT 'free',
  created_at TIMESTAMPTZ DEFAULT now()
);

-- 名刺カード
CREATE TABLE cards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id),
  slug TEXT UNIQUE NOT NULL,
  card_name TEXT, -- 「仕事用」「副業用」等
  full_name TEXT NOT NULL,
  full_name_kana TEXT,
  title TEXT, -- 肩書き
  company TEXT,
  email TEXT,
  phone TEXT,
  website TEXT,
  social_links JSONB DEFAULT '{}', -- {"x": "...", "github": "...", ...}
  bio TEXT,
  avatar_url TEXT,
  template TEXT DEFAULT 'minimal',
  custom_colors JSONB,
  is_primary BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- 名刺交換履歴
CREATE TABLE exchanges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  card_id UUID REFERENCES cards(id),
  viewer_name TEXT, -- 相手の名前（任意入力）
  viewer_email TEXT,
  viewer_info JSONB, -- 相手がDigiMeishiユーザーなら自動記録
  note TEXT, -- メモ（「渋谷のイベントで会った」等）
  follow_up_at TIMESTAMPTZ, -- フォローアップリマインダー日時
  follow_up_done BOOLEAN DEFAULT false,
  exchanged_at TIMESTAMPTZ DEFAULT now()
);

-- カードアクセスログ
CREATE TABLE card_views (
  id BIGSERIAL PRIMARY KEY,
  card_id UUID REFERENCES cards(id),
  referrer TEXT,
  user_agent TEXT,
  viewed_at TIMESTAMPTZ DEFAULT now()
);

-- サブスクリプション
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id),
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
| Vercel | $0（ISRでプロフィールページ配信、無料枠内） |
| Supabase | $0（無料枠） |
| ドメイン | $1 |
| **合計** | **~$1/月** |

### AI費用
- **$0**（AI不使用。名刺生成はテンプレートベース）

## MVPスコープ（2週間）

### Week 1
- [ ] 認証（Supabase Auth: Google + メール）
- [ ] プロフィール入力フォーム
- [ ] 名刺デザインテンプレート3種
- [ ] プロフィールページ（/[slug]）
- [ ] QRコード生成 + vCardダウンロード

### Week 2
- [ ] 交換履歴記録（アクセスログ + メモ）
- [ ] デザインテンプレート追加（計6種）
- [ ] スマホウィジェット用QR画像ダウンロード
- [ ] Stripe決済（Plus/Pro）
- [ ] LP + SEO

## マーケティング計画

### Phase 1: フリーランス層獲得（1ヶ月目）
- **X/Twitter**: 「紙の名刺、もうやめました」系の投稿でバズ狙い
- フリーランスコミュニティ（bosyu、MENTA、ランサーズ）で展開
- Product Hunt Launch
- 「デジタル名刺」「名刺 QR」のSEO対策

### Phase 2: 交流会・イベント（2-3ヶ月目）
- スタートアップ交流会でDigiMeishiを使ってみせるライブデモ
- 「名刺交換でフォロワーになる」SNS連動キャンペーン
- テックカンファレンス（技術書典、DevFest等）でのブース

### Phase 3: B2B展開（4-6ヶ月目）
- スタートアップ向けチームプラン
- Sansan/Eight連携（名刺データインポート）
- NFCカードの物販（原価$1→販売$10、利益率90%）

## リスク

| リスク | 影響 | 対策 |
|-------|------|------|
| 紙名刺文化の根強さ | 中 | 「紙+デジタル両方」の訴求、QRを名刺に印刷する提案 |
| Eight/Sansanの個人版展開 | 中 | デザイン性とUXで差別化、彼らはBtoB法人向けに注力 |
| 利用頻度の低さ | 低 | 一度設定したら常時使用。名刺は「持ち物」であり継続性高い |
| NFCカード在庫リスク | 低 | 受注生産（Alibaba等で1枚$1、2週間配送） |

## 競合分析

| 競合 | 価格 | 弱点 |
|------|------|------|
| Eight (Sansan) | 無料（法人は高額） | UIが古い、名刺管理特化でデザイン性なし |
| myBridge (LINE) | 無料 | 名刺スキャン特化、デジタル名刺ではない |
| Popl (US) | $7.99/月 | 英語のみ、日本の名刺文化に非対応 |
| HiHello (US) | $6/月 | 英語のみ |
| Canva名刺 | 無料〜$12.99/月 | デジタル名刺ではなく印刷用デザインのみ |

**DigiMeishiの差別化**: 日本の名刺文化に最適化（ふりがな、敬称、会社名上位表示）× おしゃれなデザイン × QR+NFC+Apple Wallet統合 × 月$3.2からの低価格

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | ~¥500（SNS+コミュニティ） |
| ARPU（月間） | ¥650（Plus/Pro加重平均） |
| 粗利率 | 99.2%（AI不使用） |
| LTV（18ヶ月想定） | ¥11,700 |
| LTV/CAC | 23.4x |
| 月間チャーン | ~4%（名刺は「持ち物」なので低チャーン） |
| Payback Period | 1ヶ月未満 |
