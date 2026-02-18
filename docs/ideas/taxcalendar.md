# 📆 TaxCalendar — フリーランス向け税務・届出期限リマインダー

## 概要

フリーランス・個人事業主向けに、確定申告・消費税申告・住民税・社会保険料・各種届出の期限をカレンダーで一元管理し、期限前にリマインダーを送るサービス。「気づいたら期限過ぎてて延滞税が…」を防ぐ。日本のフリーランス特化。

## ターゲット

- **メイン**: 日本のフリーランス・個人事業主（推定200万人）
- **サブ**: 副業サラリーマン（確定申告が必要な層、推定300万人）
- **ペルソナ**: 山田さん（30歳）Webデザイナーのフリーランス。独立2年目。昨年、消費税の中間申告を忘れて延滞税$50相当を支払い。税理士に頼むほどの規模ではないが、自分で管理するのは限界を感じている。

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | ¥0 | 確定申告・住民税のみ、メール通知1回 |
| Pro | ¥500/月（≈$3.3） | 全税目対応、LINE/Slack通知、14日前・7日前・3日前・当日リマインダー、カスタム届出追加 |
| Pro年払い | ¥4,800/年（≈$32/年 = $2.7/月） | Pro全機能、2ヶ月分お得 |

## ユーザーフロー

1. サインアップ（メール or LINE連携）
2. プロフィール設定: 開業年、課税区分（免税/課税）、青色/白色、業種
3. 自動でカレンダー生成（確定申告3/15、消費税3/31、住民税6月〜、予定納税7/31・11/30等）
4. 通知設定（LINE / メール / Slack、何日前に通知するか）
5. カスタム届出追加（例: 「青色申告承認申請 3/15まで」）
6. 期限前に自動リマインダー
7. 「完了」ボタンで済みマーク → 次の期限に自動フォーカス

## アーキテクチャ

```
[ユーザー] → [Next.js on Vercel]
                    ↓
            [Supabase (Auth + DB)]
                    ↓
[EventBridge (日次)] → [Lambda (リマインダーチェック)]
                    ↓
            [LINE Messaging API / SES / Slack]
```

## DB設計

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  line_user_id TEXT,
  plan TEXT DEFAULT 'free',
  -- 税務プロフィール
  business_start_year INTEGER,
  tax_type TEXT DEFAULT 'exempt', -- exempt(免税), taxable(課税)
  filing_type TEXT DEFAULT 'white', -- white(白色), blue(青色)
  has_employees BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 税務イベントマスター（システム定義）
CREATE TABLE tax_events_master (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL, -- '確定申告', '消費税中間申告'
  description TEXT,
  deadline_rule JSONB NOT NULL, -- {"month": 3, "day": 15} or {"relative": "quarter_end+2m"}
  applies_to JSONB, -- {"tax_type": ["taxable"]} 等の条件
  category TEXT -- income_tax, consumption_tax, resident_tax, insurance, custom
);

-- ユーザーのカレンダー（自動生成 + カスタム）
CREATE TABLE user_deadlines (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  tax_event_id UUID REFERENCES tax_events_master(id),
  custom_name TEXT, -- カスタム届出の場合
  deadline_date DATE NOT NULL,
  status TEXT DEFAULT 'upcoming', -- upcoming, reminded, completed, overdue
  completed_at TIMESTAMPTZ
);

-- 通知設定
CREATE TABLE notification_settings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  channel TEXT NOT NULL, -- line, email, slack
  config JSONB NOT NULL,
  days_before INTEGER[] DEFAULT '{14,7,3,1,0}',
  enabled BOOLEAN DEFAULT true
);

-- 通知履歴
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  deadline_id UUID REFERENCES user_deadlines(id),
  sent_at TIMESTAMPTZ DEFAULT NOW(),
  channel TEXT
);
```

## コスト見積もり

| 項目 | 月額 |
|------|------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| AWS Lambda (日次チェック) | $0.10 |
| LINE Messaging API | $0（Free tier: 200通/月） |
| AWS SES | $0.10（1,000通/月） |
| **合計** | **$0.20/月** (100ユーザー時) |

## MVPスコープ（2週間）

### Week 1
- 認証（Supabase Auth + LINE Login）
- 税務プロフィール設定UI
- 税務イベントマスターデータ投入（確定申告、消費税、住民税、予定納税、社会保険等 20+イベント）
- カレンダー自動生成ロジック

### Week 2
- LINE / メール通知
- カレンダービュー（直近の期限一覧）
- カスタム届出追加機能
- LP + Stripe決済

## マーケ計画

1. **SEO**: 「確定申告 期限 いつ」「フリーランス 税金 スケジュール」（検索ボリューム大）
2. **Twitter/X**: 確定申告シーズン（1-3月）に集中マーケ → 「まだ間に合う？」系ツイート
3. **note/Zenn**: 「フリーランス1年目の税務カレンダー完全版」記事 → TaxCalendar紹介
4. **フリーランスコミュニティ**: Lancers/CrowdWorks/ココナラの掲示板
5. **確定申告シーズン**: 年1回の最大獲得チャンス。1-3月に広告費集中
6. **口コミ**: 「延滞税を防げた」体験がSNSで拡散

## 技術スタック

- **フロントエンド**: Next.js 14 + Tailwind CSS + shadcn/ui
- **バックエンド**: Next.js API Routes + AWS Lambda
- **DB**: Supabase (PostgreSQL)
- **認証**: Supabase Auth (LINE Login / メール)
- **決済**: Stripe
- **通知**: LINE Messaging API, AWS SES, Slack Webhook

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| 税制改正への追従 | 高 | 年1回のマスターデータ更新。国税庁RSSで変更検知 |
| 会計ソフト（freee/マネーフォワード）が同機能追加 | 高 | 既にカレンダー機能はあるが「通知」が弱い。LINE通知特化で差別化 |
| 日本市場限定（$20 = ¥3,000） | 中 | 日本のフリーランス200万人市場で十分。将来的にUS/UK展開可能 |
| 季節性（1-3月に集中） | 中 | 年間通じた届出（住民税、社保、予定納税）でリテンション維持 |

## 競合分析

| サービス | 月額 | 特徴 | TaxCalendarの優位性 |
|----------|------|------|---------------------|
| freee | ¥1,980〜 | 会計ソフト（カレンダーは付録） | 10分の1の価格、通知特化 |
| マネーフォワード | ¥800〜 | 会計ソフト | リマインダー特化で安い |
| Googleカレンダー手動 | ¥0 | 自分で登録 | 自動生成、税制変更追従 |
| 税理士 | ¥1万〜/月 | 全部お任せ | 100分の1のコスト |

## $20達成シナリオ

```
Month 1 (確定申告シーズン): 無料ユーザー200人
Month 2: Pro 4人 = ¥2,000 ≈ $13/月
Month 3: Pro 6人 = ¥3,000 ≈ $20/月 ✅
Month 6: Pro 15人 + 年払い10人 = ¥11,500 ≈ $77/月
```

**必要有料ユーザー数: Pro 6人で$20達成（¥500×6 = ¥3,000 ≈ $20）**

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0（SEO/SNS） |
| ARPU | $3.30（¥500） |
| 粗利率 | 99.4%（$3.30 - $0.002/ユーザー） |
| LTV（12ヶ月） | $39.60 |
| LTV/CAC | ∞ |
| チャーン率（予測） | 2%/月（税務は毎年必須→極低チャーン） |
| Payback Period | 即時 |
