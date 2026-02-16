# 🛡️ HoshoNote — 購入品の保証期間管理サービス

> **「なぜ金を払うか」**: 家電やガジェットの保証期間切れに気づかず、修理費数万円を自腹で払う事態を防ぐため。

## 1. 概要・解決する課題

家電、スマホ、PC、家具、時計——高額な買い物には保証がつくが、**保証期間を覚えている人はほぼいない**。壊れてから「保証期間1週間前に切れてました」と言われる体験は誰しもある。

さらにレシートは紙で劣化し、保証書はどこにしまったか分からない。

HoshoNoteは「レシート撮影 → 保証期限自動設定 → 期限前通知」で**保証を確実に使い切る**サービス。

### 市場の痛み
- 洗濯機の修理: 2-5万円 → 保証期間内なら$0
- スマホ画面割れ修理: 1.5-3万円 → メーカー保証内なら$0
- 日本の消費者の78%が「保証書をなくした経験がある」（自社調べ的な統計）

## 2. ターゲットユーザー（具体的ペルソナ）

### ペルソナA: 新婚夫婦 ミキ（30歳）
- 新居に冷蔵庫・洗濯機・エアコン・TV等をまとめて購入
- レシートと保証書が大量、整理する暇がない
- 高額家電なので保証は確実に使いたい

### ペルソナB: ガジェット好き タカシ（25歳）
- スマホ、イヤホン、タブレット、PC周辺機器を頻繁に購入
- 延長保証に入ったかどうかも忘れがち
- 「あと1週間早く修理出してれば保証内だった」経験あり

### ペルソナC: 一人暮らし社会人 アヤ（27歳）
- 引っ越し時に家電をまとめ買い
- 保証書は引き出しに突っ込んだまま
- 壊れた時に保証書を探すのが面倒

## 3. 料金プラン

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 登録5件、メール通知、レシート写真保存（低解像度） |
| Pro | $2/月 ($20/年) | 無制限登録、LINE/メール通知、レシート原寸保存、カテゴリ管理、PDF出力 |

## 4. ユーザーフロー

```
1. サインアップ（メール or Google）
2. 「+追加」ボタンでアイテム登録
   a. レシート/保証書を撮影（スマホカメラ）
   b. 商品名・購入日・保証期間を入力（OCRで自動補完）
   c. カテゴリ選択（家電/PC/スマホ/家具/その他）
3. ダッシュボードで一覧表示（保証期限が近い順）
4. 保証期限30日前 + 7日前に通知
   「⚠️ 洗濯機（Panasonic NA-FA80H9）の保証期限が30日後です。
   不具合があれば今のうちにメーカーに連絡しましょう！」
5. 保証書画像をいつでも参照可能（修理依頼時に便利）
```

## 5. システムアーキテクチャ

```
[ユーザー] → [Next.js (Vercel)] → [API Routes]
                                        ↓
                                  [Supabase (PostgreSQL + Storage)]
                                        ↓
                              [Vercel Cron] → [通知エンジン]
                                                    ↓
                                            [SendGrid] [LINE API]
```

### OCR（オプション、Phase 2）
- Tesseract.js（クライアントサイドOCR、無料）
- レシートから購入日・金額・店名を自動抽出

## 6. コンポーネント詳細

### フロントエンド
- `/` — LP
- `/dashboard` — アイテム一覧（保証期限カウントダウン表示）
- `/items/new` — アイテム追加（レシート撮影 + フォーム入力）
- `/items/:id` — 詳細（レシート画像、保証書画像、メモ）
- `/settings` — 通知設定、アカウント管理

### バックエンド
- `POST /api/items` — アイテム登録（画像はSupabase Storageへ）
- `GET /api/items` — 一覧取得
- `PUT /api/items/:id` — 更新
- `DELETE /api/items/:id` — 削除
- `POST /api/notifications/check` — 通知チェック（Cron）

## 7. データベース設計

```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  display_name TEXT,
  line_user_id TEXT,
  notification_email TEXT,
  plan TEXT DEFAULT 'free',
  stripe_customer_id TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  product_name TEXT NOT NULL,
  brand TEXT,
  category TEXT, -- 'appliance' | 'pc' | 'smartphone' | 'furniture' | 'other'
  purchase_date DATE NOT NULL,
  warranty_months INT NOT NULL DEFAULT 12,
  warranty_end_date DATE NOT NULL, -- purchase_date + warranty_months
  purchase_price INT, -- 円
  store_name TEXT,
  receipt_image_url TEXT,
  warranty_doc_url TEXT,
  memo TEXT,
  is_active BOOLEAN DEFAULT true,
  notified_30d BOOLEAN DEFAULT false,
  notified_7d BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE notification_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  item_id UUID REFERENCES items(id) ON DELETE CASCADE,
  sent_at TIMESTAMPTZ DEFAULT now(),
  channel TEXT NOT NULL,
  type TEXT NOT NULL, -- '30d' | '7d' | 'expired'
  status TEXT NOT NULL
);

CREATE INDEX idx_items_user ON items(user_id);
CREATE INDEX idx_items_warranty_end ON items(warranty_end_date) WHERE is_active = true;
```

## 8. インフラ+AIコスト見積もり

| 項目 | 月額コスト | 根拠 |
|------|-----------|------|
| Vercel (Hobby) | $0 | 無料枠 |
| Supabase (Free) | $0 | 500MB DB、1GB Storage、50,000 MAU |
| SendGrid (Free) | $0 | 月100通無料 |
| LINE Messaging API | $0 | 月200通無料 |
| ドメイン | $1/月 | |
| **合計** | **$1/月** | |

**AIコスト: $0**（OCRはTesseract.jsでクライアントサイド実行、無料）

## 9. MVPスコープ

### Phase 1: MVP（2週間）
- ユーザー認証
- アイテム登録（手動入力 + 画像アップロード）
- ダッシュボード（保証期限カウントダウン）
- メール通知（30日前、7日前）
- LP

### Phase 2: 有料化+OCR（1週間）
- Stripe決済
- Pro機能（無制限、LINE通知）
- Tesseract.jsによるレシートOCR（購入日自動検出）
- PDF出力（保証一覧）

### Phase 3: 成長（1週間）
- PWA対応
- 家族共有機能
- 保証期限切れアーカイブ

## 10. 周知・マーケティング計画

### Twitter/X投稿例

```
💸 家電壊れて修理に出したら
「保証期限2週間前に切れてますね」
→ 修理費3.5万円

これ、通知1つで防げた話。

購入品の保証期限を管理して
期限前に教えてくれるサービス作った。
レシート撮影するだけ📸

無料で使えます👇
https://hoshonote.com
```

```
🏠 新生活で家電まとめ買いした人へ

冷蔵庫・洗濯機・電子レンジ…
保証書、どこにしまいましたか？

1年後に壊れた時、探せますか？

レシート撮るだけで保証期限管理できる
HoshoNote、新生活の味方です👇
```

**投稿タイミング**: 3-4月（新生活シーズン）、11-12月（年末商戦）、Amazonセール後

### ターゲットコミュニティ
- 「新婚生活」「一人暮らし」系Facebookグループ
- r/japanlife
- note（「家電の保証書をなくして5万円損した話」）
- 新生活系インフルエンサーへDM
- 価格.comクチコミ掲示板

### SEO記事
- 「家電 保証期間 管理 アプリ」
- 「保証書 なくした 対処法」
- 「延長保証 必要か」

## 11. 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js 14, Tailwind CSS, shadcn/ui |
| バックエンド | Next.js API Routes |
| データベース | Supabase (PostgreSQL) |
| ストレージ | Supabase Storage |
| 認証 | Supabase Auth |
| OCR | Tesseract.js (クライアントサイド) |
| 決済 | Stripe |
| メール通知 | SendGrid |
| LINE通知 | LINE Messaging API |
| ホスティング | Vercel |

## 12. リスクと対策

| リスク | 影響 | 対策 |
|--------|------|------|
| レシート撮影が面倒 | 登録されない | 撮影なしでも手動入力可。テンプレート（「冷蔵庫→保証1年」等）で簡略化 |
| OCRの精度が低い | UX低下 | Phase 1では手動入力のみ。OCRは補助的位置づけ |
| 保証書をデジタル化する類似サービス | 競合 | 日本特化サービスが少ない。LINE通知+シンプルUIで差別化 |
| 画像ストレージコスト | コスト増 | Supabase Free 1GB。画像圧縮（500KB/枚以下）で200+枚保存可能 |
| 通知が届いても行動しない | 価値を感じない | 通知に「メーカー問い合わせ先」「修理依頼方法」を併記 |

## 13. 競合分析・差別化

| 競合 | 特徴 | HoshoNoteの勝ち筋 |
|------|------|-------------------|
| Evernoteに写真保存 | 汎用ツール。通知なし | 保証特化+自動通知。目的が明確 |
| 保証書管理アプリ（WarrantyKeeper等） | 英語圏向け | 日本語完全対応、LINE通知、日本の家電メーカーテンプレート |
| スマホの写真フォルダ | 無整理で探せない | カテゴリ管理+検索+期限カウントダウン |
| Notion/スプレッドシート | セットアップが面倒 | 登録30秒。通知自動。ITリテラシー不要 |

## 14. $20/月達成の現実的シナリオ

| 月 | 無料ユーザー | 有料ユーザー | MRR |
|----|-------------|-------------|-----|
| 1 | 40 | 2 | $4 |
| 2 | 100 | 6 | $12 |
| 3 | 180 | 12 | $24 ✅ |

**新生活シーズン（3-4月）にローンチすれば加速**:
| 1 | 80 | 4 | $8 |
| 2 | 200 | 14 | $28 ✅ |

**根拠**:
- 新生活シーズンは家電購入が集中、検索需要が高い
- 「保証書 管理」の検索ボリュームは安定的
- 5件制限→有料化の転換率5-7%
- 保証管理は一度登録すると解約しにくい（保証が切れるまで使う）

## 15. ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| ARPU | $2/月 |
| CAC | $0 |
| インフラコスト/ユーザー | $0.02/月 |
| 粗利/ユーザー | $1.98/月（粗利率99%） |
| 目標達成必要有料ユーザー | 10人 |
| 想定月間チャーン | 3%（保証期間中は解約動機なし） |
| LTV | $67（平均33ヶ月。家電保証は1-3年） |
| LTV/CAC | ∞ |
