# 💳 SubsAlert — サブスク解約忘れ防止リマインダー

> **「なぜ金を払うか」**: 解約し忘れたサブスクで毎月お金が消えるのを防ぐため。月額$2で年間数万円の無駄を防止。

## 1. 概要・解決する課題

Netflix、Spotify、Adobe、ジム、新聞、アプリ内課金…現代人は平均12個のサブスクを契約し、うち3-4個は「使っていないのに払い続けている」状態。日本では年間平均2.4万円が無駄なサブスクに消えている（MMD研究所調べ）。

SubsAlertは「登録→更新日前にリマインド→不要なら解約」というシンプルなフローで、**解約忘れによるお金の無駄をゼロにする**。

### 既存の家計簿アプリとの違い
- 家計簿アプリ: 使ったお金を「記録」する（事後）
- SubsAlert: これから引き落とされるお金を「防ぐ」（事前）
- サブスク特化 → 登録3秒、通知の精度が高い

## 2. ターゲットユーザー（具体的ペルソナ）

### ペルソナA: 大学生 ユウキ（21歳）
- 無料トライアルを片っ端から登録、解約を忘れて毎月3,000円無駄に
- スマホ中心の生活、アプリのUXに敏感
- バイト代で生活しているので出費に敏感

### ペルソナB: 会社員 サトミ（34歳・既婚）
- 夫婦でNetflix、Hulu、Disney+、Amazon Prime、Spotifyを契約
- 子供の習い事のオンライン教材も複数
- 家計を見直したいが、何を契約しているか全体像が分からない

### ペルソナC: フリーランスデザイナー ケンタ（28歳）
- Adobe、Figma、Canva Pro、ストックフォト、クラウドストレージ等
- 案件ごとに短期で使うサービスを登録し、解約を忘れる
- 確定申告時に「これ何のサブスク？」と困る

## 3. 料金プラン

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | サブスク5件まで登録、メール通知のみ |
| Pro | $2/月 ($20/年) | 無制限登録、LINE/メール/Push通知、カテゴリ管理、月次レポート |

**価格の根拠**: 1つの解約忘れ（平均月1,000円）を防ぐだけで月$2は即回収。ROIが明確。

## 4. ユーザーフロー

```
1. メールアドレスでサインアップ（10秒）
2. サブスクを追加（サービス名・金額・更新日・周期を入力）
   - よく使うサービスはテンプレートから選択（Netflix等50+）
3. 通知設定（更新日の何日前に通知するか。デフォルト3日前）
4. 更新日前にメール/LINE通知
   「📢 あと3日でNetflix（月額1,490円）が更新されます。継続しますか？」
5. ダッシュボードで月額合計・年額合計を確認
6. 月次レポートメール（今月の支払い予定サマリー）
```

## 5. システムアーキテクチャ

```
[ユーザー] → [Next.js (Vercel)] → [API Routes]
                                        ↓
                                  [Supabase (PostgreSQL)]
                                        ↓
                              [Cron (Vercel Cron)] → [通知エンジン]
                                                        ↓
                                                [SendGrid] [LINE API]
```

### 選定理由
- **Vercel**: フロントもAPIも無料枠で運用可能。Cronも無料（1日1回）
- **Supabase**: PostgreSQL + Auth無料枠（50,000 MAU、500MB DB）
- **SendGrid**: 月100通無料、その後も低コスト
- **LINE Messaging API**: 月200通無料（Push）

## 6. コンポーネント詳細

### フロントエンド (Next.js App Router)
- `/` — LP（サービス説明+登録CTA）
- `/dashboard` — サブスク一覧、月額/年額サマリー
- `/add` — サブスク追加（テンプレート選択 or 手動入力）
- `/settings` — 通知設定、LINE連携、アカウント管理

### バックエンド (Next.js API Routes)
- `POST /api/subscriptions` — サブスク登録
- `GET /api/subscriptions` — 一覧取得
- `PUT /api/subscriptions/:id` — 更新
- `DELETE /api/subscriptions/:id` — 削除
- `POST /api/notifications/check` — Cronから呼ばれ、通知対象を抽出して送信
- `POST /api/line/webhook` — LINE連携用

### 通知エンジン
- 毎日09:00 JSTにCron実行
- 各ユーザーのサブスクを走査、通知対象を抽出
- メール or LINE で通知送信

## 7. データベース設計

```sql
-- ユーザー（Supabase Auth利用）
-- auth.usersテーブルを利用

-- プロフィール
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  display_name TEXT,
  line_user_id TEXT,
  notification_email TEXT,
  notification_days_before INT DEFAULT 3,
  plan TEXT DEFAULT 'free', -- 'free' | 'pro'
  stripe_customer_id TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- サブスクリプション
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  service_name TEXT NOT NULL,
  amount INT NOT NULL, -- 円単位
  currency TEXT DEFAULT 'JPY',
  billing_cycle TEXT NOT NULL, -- 'monthly' | 'yearly'
  next_billing_date DATE NOT NULL,
  category TEXT, -- 'entertainment' | 'productivity' | 'education' | 'other'
  memo TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- 通知履歴
CREATE TABLE notification_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subscription_id UUID REFERENCES subscriptions(id) ON DELETE CASCADE,
  sent_at TIMESTAMPTZ DEFAULT now(),
  channel TEXT NOT NULL, -- 'email' | 'line'
  status TEXT NOT NULL -- 'sent' | 'failed'
);

-- インデックス
CREATE INDEX idx_subscriptions_user ON subscriptions(user_id);
CREATE INDEX idx_subscriptions_next_billing ON subscriptions(next_billing_date) WHERE is_active = true;
```

## 8. インフラ+AIコスト見積もり

| 項目 | 月額コスト | 根拠 |
|------|-----------|------|
| Vercel (Hobby) | $0 | 個人プロジェクト無料枠（100GB帯域、Cron 1回/日） |
| Supabase (Free) | $0 | 50,000 MAU、500MB DB、2GB帯域 |
| SendGrid (Free) | $0 | 月100通まで無料（初期は十分） |
| LINE Messaging API | $0 | 月200通まで無料 |
| ドメイン (.com) | $1/月 | 年$12 |
| **合計** | **$1/月** | |

**AIコスト: $0**（AI機能不使用。ルールベースの通知のみ）

**スケール時（有料ユーザー50人）**:
- Supabase Pro: $25/月 → ユーザー50人×$2 = $100/月収入。粗利75%
- SendGrid: 50人×4通/月 = 200通 → 無料枠内

## 9. MVPスコープ

### Phase 1: MVP（2週間）
- ユーザー認証（Supabase Auth）
- サブスク登録・一覧・編集・削除
- テンプレート50件（Netflix, Spotify, Amazon Prime等）
- メール通知（更新日前）
- ダッシュボード（月額合計表示）
- LP

### Phase 2: 有料化（1週間）
- Stripe決済連携
- Pro機能（無制限登録、LINE通知）
- 月次レポートメール

### Phase 3: 成長（1週間）
- PWA対応（ホーム画面追加）
- カテゴリ別分析
- 年間支出グラフ

## 10. 周知・マーケティング計画

### Week 1-2: SNS投稿（無料）

**Twitter/X投稿例**:
```
📢 サブスク、いくつ契約してるか即答できますか？

日本人の平均：12個契約、うち3-4個は使ってない
→ 年間2.4万円が消えてる計算

「解約忘れ」だけで年間2.4万円損するの、
もうやめませんか？

SubsAlert作りました。無料で使えます👇
https://subsalert.com
```

```
💸 先月、使ってないジムのサブスク代1万円引き落とされてた…

こういう「気づいたら引き落とし」を
更新3日前に通知してくれるサービス作った。

無料で5件まで登録できます。
まず自分のサブスク全部洗い出してみて👇
```

**投稿タイミング**: 月初（クレカ明細確認タイミング）、給料日前

### Week 3-4: コミュニティ投稿

**ターゲットコミュニティ**:
- r/japanlife、r/japanfinance（Reddit）
- 「家計管理」「節約」系Facebookグループ
- note（「サブスク地獄から脱出した話」記事）
- 節約系YouTuberへのDM（レビュー依頼）

**note記事タイトル例**:
「サブスクに年間5万円無駄にしていた話と、それを0にした方法」

### Month 2: 口コミ促進
- 「友達紹介で1ヶ月Pro無料」キャンペーン
- 「サブスク断捨離チャレンジ」ハッシュタグ企画

## 11. 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js 14 (App Router), Tailwind CSS, shadcn/ui |
| バックエンド | Next.js API Routes |
| データベース | Supabase (PostgreSQL) |
| 認証 | Supabase Auth |
| 決済 | Stripe |
| メール通知 | SendGrid |
| LINE通知 | LINE Messaging API |
| ホスティング | Vercel |
| Cron | Vercel Cron Jobs |

## 12. リスクと対策

| リスク | 影響 | 対策 |
|--------|------|------|
| 「スマホのカレンダーで十分」と思われる | 利用されない | カレンダーにはない月額合計・年額合計の可視化、テンプレートによる簡便さを訴求 |
| マネーフォワード等の大手と競合 | ユーザー獲得困難 | サブスク「だけ」に特化。登録10秒。大手アプリの銀行連携は不要 |
| 通知が多すぎてスルーされる | 解約防止効果なし | 月1回の「まとめ通知」モード実装。通知頻度をユーザーが制御 |
| Supabase無料枠超過 | コスト増 | 50人有料ユーザー到達後にPro移行。その時点で月収$100 |
| 解約忘れが少ない人には価値がない | 解約率高い | 無料プランで継続利用。月次レポートの「節約額」表示で価値を実感 |

## 13. 競合分析・差別化

| 競合 | 特徴 | SubsAlertの勝ち筋 |
|------|------|-------------------|
| マネーフォワード ME | 総合家計簿。銀行連携で自動取得 | 機能が多すぎて使いこなせないライト層向け。サブスク特化でシンプル |
| Subscriptions (iOS) | サブスク管理アプリ。買い切り$5 | Web対応でデバイス不問。LINE通知。無料プランあり |
| Bobby (iOS) | 美しいUIのサブスク管理 | 日本のサービステンプレート対応。LINE通知。日本語完全対応 |
| スマホ設定のサブスク管理 | Apple/Google標準 | App Store/Google Play経由のみ。Webサービス（ジム、新聞等）は管理不可 |

**勝ち筋**: 日本市場特化（LINE通知、日本のサービステンプレート、円表示）+ Web対応 + 無料で始められる

## 14. $20/月達成の現実的シナリオ

| 月 | 無料ユーザー | 有料ユーザー | MRR |
|----|-------------|-------------|-----|
| 1 | 50 | 2 | $4 |
| 2 | 120 | 8 | $16 |
| 3 | 200 | 15 | $30 ✅ |

**根拠**:
- Twitter投稿で月50-70の新規登録（サブスク管理は共感されやすいテーマ）
- 無料→有料転換率: 5-8%（5件制限が自然なアップセル）
- note記事がSEOで継続流入
- Month 2の$16で未達だが、紹介キャンペーンで加速。Month 3で$30到達

## 15. ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| ARPU | $2/月 |
| CAC | $0（オーガニック獲得） |
| インフラコスト/ユーザー | $0.02/月（Supabase Free時） |
| 粗利/ユーザー | $1.98/月（粗利率99%） |
| 目標達成必要有料ユーザー | 10人 |
| 想定月間チャーン | 5%（解約忘れ防止の実績が出れば低チャーン） |
| LTV | $40（平均20ヶ月利用） |
| LTV/CAC | ∞（CAC≈$0） |
