# 🎯 PitchCraft — AIピッチデッキ自動生成SaaS

## 概要

ビジネスアイデアをテキストで入力するだけで、投資家・コンペ向けのプロフェッショナルなピッチデッキ（プレゼン資料）を自動生成するSaaS。海外ではGamma.appが$25M+ ARR規模で成功しているが、日本語に最適化されたピッチデッキ特化ツールは存在しない。

## 海外事例分析

| サービス | 規模 | 特徴 |
|---------|------|------|
| **Gamma.app** (US) | $25M+ ARR, $40M調達 | AI汎用プレゼン生成。ピッチ特化ではない |
| **Pitch** (DE) | $135M調達 | コラボレーション重視のプレゼンツール |
| **Beautiful.ai** (US) | $29M調達 | AIデザインアシストのプレゼンツール |
| **Slidebean** (US) | $5M+ ARR | ピッチデッキ特化。テンプレ+AIコンテンツ |
| **Tome** (US) | $75M調達 | AIストーリーテリング型プレゼン |

**日本の現状**: パワポかCanvaで手作り。日本語フォント・レイアウトに最適化されたAIツールなし。スタートアップ資金調達額は年々増加（2024年: 約8,000億円）し、ピッチ機会も急増中。

## ターゲット

### プライマリ
- **スタートアップ創業者**（シード〜シリーズA）
- ピッチイベント参加者（IVS、B Dash Camp、JAFCO等）
- 1人〜3人の初期チームでデザイナー不在

### セカンダリ
- **ビジネスコンテスト参加者**（学生起業、社内新規事業）
- **フリーランス・コンサル**（提案書作成）
- **中小企業**（銀行融資・補助金申請用資料）

## 料金プラン

| プラン | 月額 | 内容 |
|-------|------|------|
| **Free** | ¥0 | 月1デッキ、基本テンプレ3種、PitchCraftロゴ入り、PDF出力 |
| **Starter** | ¥980（~$6.5） | 月5デッキ、テンプレ15種、ロゴなし、PDF/PPTX出力、AIリライト |
| **Pro** | ¥1,980（~$13） | 無制限デッキ、全テンプレ、カスタムブランディング、投資家フィードバックAI、PPTX/Keynote出力 |

### $20達成シナリオ
- **Starter 2人 + Pro 1人** = $13 + $13 = $26/月
- **達成時期**: 2ヶ月目（スタートアップコミュニティへの直接リーチ）

## ユーザーフロー

```
1. LPにアクセス → Google/メール登録
2. 「何のビジネスですか？」→ テキスト入力（200字程度）
3. AIがインタラクティブに質問（市場規模、差別化、チーム等）
4. 10秒でピッチデッキ（10〜15スライド）を自動生成
5. プレビュー → スライドごとに編集・AIリライト
6. テンプレート変更（ワンクリックで全体のデザイン切替）
7. PDF / PPTX / Keynote でエクスポート
8. 【Pro】投資家目線AIフィードバック（弱い箇所の指摘）
```

## デザインコンセプト

### ビジュアルアイデンティティ
- **カラー**: インディゴ（信頼感）× ホワイト × アクセントにオレンジ
- **フォント**: Noto Sans JP（本文）+ Inter（英数字）
- **テーマ**: 「プロフェッショナル × クリーン × 自信」

### ピッチデッキテンプレート
- **VC Standard**: YC/500 Startups風の王道10枚構成
- **Japan VC**: 日本のVC好みの市場分析重視スタイル
- **Demo Day**: インパクト重視、ビジュアル大きめ
- **Minimal**: モノクロ、データ重視、B2B向け
- **Bold**: 大胆な色使い、D2C/コンシューマー向け
- **Corporate**: 大企業内新規事業、稟議書風

### スライド構成（自動生成）
1. タイトル / 会社名
2. 課題（Problem）
3. 解決策（Solution）
4. プロダクト（デモ/スクショ）
5. 市場規模（TAM/SAM/SOM）
6. ビジネスモデル
7. トラクション
8. 競合比較
9. チーム
10. 資金計画 / Ask

## アーキテクチャ

```
[ユーザー入力] → [Next.js Frontend (Vercel)]
                      ↓
              [API Routes]
                      ↓
              [GPT-4o-mini] → コンテンツ生成
                      ↓
              [スライドレンダラー (React + SVG/Canvas)]
                      ↓
              [Supabase PostgreSQL] → 保存
                      ↓
              [PDF生成 (puppeteer/playwright)] → エクスポート
              [PPTX生成 (pptxgenjs)] → エクスポート
```

### 技術スタック
- **Frontend**: Next.js 14 + Tailwind CSS + Framer Motion
- **Backend**: Next.js API Routes (Vercel Serverless)
- **DB**: Supabase (PostgreSQL + Auth + Storage)
- **AI**: GPT-4o-mini（コンテンツ生成、リライト、フィードバック）
- **スライドレンダリング**: React + CSS-in-JS → HTML/CSS → PDF
- **PPTX出力**: pptxgenjs (OSS)
- **ホスティング**: Vercel
- **決済**: Stripe

## DB設計

```sql
-- ユーザー
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  display_name TEXT,
  company TEXT,
  plan TEXT DEFAULT 'free',
  decks_this_month INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- ピッチデッキ
CREATE TABLE decks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  business_description TEXT,
  template TEXT DEFAULT 'vc-standard',
  brand_colors JSONB,
  is_public BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- スライド
CREATE TABLE slides (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deck_id UUID REFERENCES decks(id) ON DELETE CASCADE,
  slide_type TEXT NOT NULL, -- problem, solution, market, etc.
  title TEXT,
  content JSONB NOT NULL, -- 構造化されたスライドコンテンツ
  notes TEXT, -- スピーカーノート
  sort_order INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- AIフィードバック（Pro用）
CREATE TABLE feedbacks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deck_id UUID REFERENCES decks(id) ON DELETE CASCADE,
  slide_id UUID REFERENCES slides(id),
  category TEXT, -- clarity, data, storytelling, design
  score INT, -- 1-10
  suggestion TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
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
| Vercel | $0（無料枠） |
| Supabase | $0（無料枠） |
| GPT-4o-mini | ~$5（デッキ生成: 1デッキ$0.03 × 月150デッキ） |
| ドメイン | $1 |
| **合計** | **~$6/月** |

### AI費用詳細
- デッキ1つ（10スライド）の生成: ~$0.03（GPT-4o-mini、約3000トークン入力 + 5000トークン出力）
- AIリライト（1スライド）: ~$0.002
- 投資家フィードバック: ~$0.01/デッキ
- 月間50デッキ生成想定: $1.50 + リライト$0.50 + フィードバック$0.50 = ~$2.50

## MVPスコープ（2週間）

### Week 1
- [ ] 認証（Supabase Auth）
- [ ] ビジネス説明入力 → AIでスライドコンテンツ生成
- [ ] スライドレンダラー（HTML/CSS、テンプレ2種）
- [ ] 基本編集（テキスト編集、順序変更）
- [ ] PDF出力

### Week 2
- [ ] テンプレート追加（計4種）
- [ ] AIリライト機能
- [ ] PPTX出力（pptxgenjs）
- [ ] Stripe決済
- [ ] LP + SEO

## マーケティング計画

### Phase 1: コミュニティ浸透（1ヶ月目）
- **スタートアップ系Slack/Discord**に投稿（Startup Hub Tokyo、ICCコミュニティ）
- X/Twitterで「3分でピッチデッキ完成」のデモ動画投稿
- **無料テンプレート配布**でSEO流入（「ピッチデッキ テンプレート」月間3,000検索）
- Product Hunt Launch

### Phase 2: イベント連動（2-3ヶ月目）
- ビジネスコンテスト/ピッチイベントのスポンサー（無料プラン提供）
- 起業家向けメディア（BRIDGE、TechCrunch Japan）でのPR
- 大学の起業部/ビジネスサークルへの無料提供

### Phase 3: B2B展開（4-6ヶ月目）
- アクセラレーター/VCとの提携（ポートフォリオ企業に推奨）
- 補助金申請向けテンプレート追加（ものづくり補助金、事業再構築補助金）
- 銀行融資向け事業計画書テンプレート

## リスク

| リスク | 影響 | 対策 |
|-------|------|------|
| Gamma.appの日本語対応 | 高 | ピッチデッキ特化+日本VC文化対応で差別化 |
| AI生成コンテンツの品質 | 中 | 編集UIの充実、テンプレート品質向上 |
| 利用頻度の低さ | 中 | 提案書/企画書にも使えるよう拡張 |
| PPTX変換の品質 | 低 | pptxgenjsの制約を把握し、対応デザインに限定 |

## 競合分析

| 競合 | 価格 | 弱点 |
|------|------|------|
| Gamma.app | $10/月 | 汎用プレゼン、日本VC文化に非対応 |
| Canva | 無料〜$12.99/月 | ピッチ特化でない、AI生成なし |
| PowerPoint | Office365 $6.99/月 | 手作業、テンプレ古い |
| Slidebean | $29/月 | 英語のみ、高額 |

**PitchCraftの差別化**: 日本のVC/補助金文化に完全対応 × ピッチデッキ特化AI × 月$6.5からの低価格

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | ~¥1,000（コミュニティ+SEO） |
| ARPU（月間） | ¥1,320（Starter/Pro加重平均） |
| 粗利率 | 93.8% |
| LTV（8ヶ月想定） | ¥10,560 |
| LTV/CAC | 10.6x |
| 月間チャーン | ~12%（利用頻度低いため高め） |
| Payback Period | 1ヶ月未満 |
