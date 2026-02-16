# 📅 YoyakuRemind — 予約キャンセル料を防ぐリマインダー

> **「なぜ金を払うか」**: 美容院・病院・飲食店の予約を忘れてキャンセル料を取られたり、無断キャンセルでブラックリスト入りするのを防ぐため。

## 1. 概要・解決する課題

美容院、歯医者、皮膚科、レストラン、整体、ネイルサロン——現代人は常に複数の予約を抱えている。しかし**予約の管理は驚くほど雑**で、以下の問題が頻発する：

- 予約を忘れて無断キャンセル → キャンセル料3,000-10,000円
- ダブルブッキング → どちらかをキャンセル
- 「あれ、次の歯医者いつだっけ？」→ 確認の電話が面倒

Googleカレンダーに手動で入れればいいが、**多くの人はそれすらしない**。

YoyakuRemindは「予約したらサッと登録 → 前日と当日朝にリマインド」の超シンプル予約管理。テンプレートから3タップで登録完了。

## 2. ターゲットユーザー（具体的ペルソナ）

### ペルソナA: OL ナナミ（26歳）
- 美容院（月1）、ネイル（月1）、歯医者（月1）、皮膚科（不定期）
- Googleカレンダーは仕事用。プライベート予約は頭の中
- 先月ネイルの予約を忘れてキャンセル料3,000円

### ペルソナB: 子育てママ リエ（35歳）
- 子供の小児科、自分の歯医者、夫の健康診断、家族のインフルエンザ予防接種
- 予約が多すぎて管理しきれない
- 子供の予防接種スケジュールを忘れそうになる

### ペルソナC: 営業マン コウジ（33歳）
- 得意先との会食予約、接待ゴルフ予約、散髪
- 仕事のスケジュールが変わって予約をリスケ忘れ
- 無断キャンセルで常連の店との関係悪化

## 3. 料金プラン

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 月5件登録、メール通知（前日のみ） |
| Pro | $2/月 ($20/年) | 無制限登録、LINE/メール通知、前日+当日朝+2時間前通知、定期予約、家族共有 |

## 4. ユーザーフロー

```
1. サインアップ（Google or メール）
2. 予約を追加
   a. カテゴリ選択: 美容院 / 病院 / 飲食店 / その他
   b. テンプレート選択: 「美容院」→ 店名、日時、メニューを入力
   c. キャンセル期限設定（任意）: 「前日18時までにキャンセルしないとキャンセル料発生」
3. ダッシュボードで今週/今月の予約一覧
4. 通知タイミング
   - キャンセル期限の前（「明日18時がキャンセル期限です。予定は大丈夫？」）
   - 前日20時（「明日10:00に○○美容院の予約があります」）
   - 当日朝8時（「今日10:00に○○美容院です。お忘れなく！」）
   - 2時間前（Pro）
5. 予約完了後に「完了」マーク or 次回予約登録
```

## 5. システムアーキテクチャ

```
[ユーザー] → [Next.js (Vercel)] → [API Routes]
                                        ↓
                                  [Supabase (PostgreSQL)]
                                        ↓
                              [Vercel Cron] → [通知エンジン]
                                                    ↓
                                            [SendGrid] [LINE API]
```

## 6. コンポーネント詳細

### フロントエンド
- `/` — LP
- `/dashboard` — 予約一覧（カレンダー表示 + リスト表示切替）
- `/add` — 予約追加（テンプレート＋フォーム）
- `/settings` — 通知設定、LINE連携

### バックエンド
- `POST /api/reservations` — 予約登録
- `GET /api/reservations` — 一覧取得
- `PUT /api/reservations/:id` — 更新
- `DELETE /api/reservations/:id` — 削除
- `POST /api/notifications/check` — 通知対象チェック+送信（Cron）

### テンプレート
```json
{
  "templates": [
    {
      "category": "beauty",
      "name": "美容院",
      "fields": ["店名", "メニュー", "担当者"],
      "default_cancel_hours": 24
    },
    {
      "category": "medical",
      "name": "歯医者",
      "fields": ["医院名", "診療内容"],
      "default_cancel_hours": null
    },
    {
      "category": "restaurant",
      "name": "レストラン",
      "fields": ["店名", "人数", "コース"],
      "default_cancel_hours": 48
    }
  ]
}
```

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

CREATE TABLE reservations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  category TEXT NOT NULL, -- 'beauty' | 'medical' | 'restaurant' | 'other'
  place_name TEXT NOT NULL,
  detail TEXT, -- メニュー、診療内容等
  reservation_date DATE NOT NULL,
  reservation_time TIME,
  cancel_deadline TIMESTAMPTZ, -- キャンセル期限
  cancel_fee INT, -- キャンセル料（円）
  is_recurring BOOLEAN DEFAULT false,
  recurring_interval TEXT, -- 'monthly' | 'weekly'
  memo TEXT,
  status TEXT DEFAULT 'upcoming', -- 'upcoming' | 'completed' | 'cancelled'
  notified_cancel_deadline BOOLEAN DEFAULT false,
  notified_day_before BOOLEAN DEFAULT false,
  notified_morning BOOLEAN DEFAULT false,
  notified_2h_before BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_reservations_user ON reservations(user_id);
CREATE INDEX idx_reservations_date ON reservations(reservation_date) WHERE status = 'upcoming';
CREATE INDEX idx_reservations_cancel ON reservations(cancel_deadline) WHERE cancel_deadline IS NOT NULL AND notified_cancel_deadline = false;
```

## 8. インフラ+AIコスト見積もり

| 項目 | 月額コスト | 根拠 |
|------|-----------|------|
| Vercel (Hobby) | $0 | 無料枠 |
| Supabase (Free) | $0 | 無料枠 |
| SendGrid (Free) | $0 | 月100通無料 |
| LINE Messaging API | $0 | 月200通無料 |
| ドメイン | $1/月 | |
| **合計** | **$1/月** | |

**AIコスト: $0**（ルールベース通知のみ）

### スケール時
- 有料ユーザー50人: 1人あたり月8通（前日+当日+キャンセル期限）×50 = 400通 → SendGrid有料プラン$15/月
- 収入: 50×$2 = $100/月。粗利$84（84%）

## 9. MVPスコープ

### Phase 1: MVP（2週間）
- ユーザー認証
- 予約登録（テンプレート+手動）
- 予約一覧（リスト表示）
- メール通知（前日、当日朝）
- LP

### Phase 2: 有料化+キャンセル期限（1週間）
- Stripe決済
- キャンセル期限通知
- LINE通知（Pro）
- 2時間前通知（Pro）

### Phase 3: 成長（1週間）
- カレンダー表示
- 定期予約（毎月の美容院等）
- 家族共有
- PWA対応

## 10. 周知・マーケティング計画

### Twitter/X投稿例

```
😱 予約忘れてキャンセル料5,000円取られた…

美容院、歯医者、レストラン…
予約って意外と管理できてなくないですか？

前日と当日朝にリマインドしてくれる
予約管理サービス作りました。

3タップで登録、あとは通知が来るのを待つだけ。
無料で使えます👇
https://yoyakuremind.com
```

```
📅 突然ですがクイズ

Q: あなたが今入れている予約、全部言えますか？

美容院、歯医者、飲食店、整体…

言えなかった人、
無断キャンセルでブラックリスト入りする前に👇
```

**投稿タイミング**: 金曜夜（週末の予約を思い出すタイミング）、月初

### ターゲットコミュニティ
- 「美容院好き」「グルメ」系Instagram/Twitterコミュニティ
- r/japanlife
- note（「予約忘れでキャンセル料3万円払った話」）
- 美容系インフルエンサーへのDM
- ホットペッパービューティーのクチコミユーザー層

### SEO記事
- 「予約忘れ 防止 アプリ」
- 「キャンセル料 避ける方法」
- 「美容院 予約 管理」

## 11. 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js 14, Tailwind CSS, shadcn/ui |
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
| 「Googleカレンダーで十分」 | 不要と判断 | キャンセル期限管理、テンプレート登録、キャンセル料表示がGCalにない価値 |
| 予約の手動登録が面倒 | 登録されない | テンプレート3タップ、よく行く店のお気に入り登録で簡略化 |
| 通知が多すぎる | 通知をOFFにされる | 通知タイミングをカスタマイズ可能。デフォルトは前日+当日の2回 |
| ホットペッパー等の予約サイトが自前リマインド | 不要に | 全予約を一元管理する価値。サイトごとにバラバラの通知を統合 |

## 13. 競合分析・差別化

| 競合 | 特徴 | YoyakuRemindの勝ち筋 |
|------|------|----------------------|
| Googleカレンダー | 汎用。キャンセル期限管理なし | 予約特化。キャンセル期限+キャンセル料表示が独自価値 |
| ホットペッパー等のリマインド | そのサイト内の予約のみ | 全予約を一元管理。複数サイトの予約を統合 |
| TimeTree | 家族カレンダー。予約特化ではない | 予約テンプレート、キャンセル期限管理に特化 |
| リマインダーアプリ | 汎用。毎回手動設定 | テンプレートで3タップ登録。予約に最適化された通知タイミング |

## 14. $20/月達成の現実的シナリオ

| 月 | 無料ユーザー | 有料ユーザー | MRR |
|----|-------------|-------------|-----|
| 1 | 60 | 3 | $6 |
| 2 | 140 | 10 | $20 ✅ |

**根拠**:
- 「予約忘れ」「キャンセル料」は共感度が高いテーマ。Twitter拡散しやすい
- 月5件制限→予約が多い人（ペルソナA, B）は自然に有料化
- Free→Pro転換率7-10%
- 美容・グルメ系コミュニティは母数が大きい
- キャンセル料を1回防ぐだけで$2/月の元が取れる（ROIが明確）

## 15. ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| ARPU | $2/月 |
| CAC | $0 |
| インフラコスト/ユーザー | $0.02/月 |
| 粗利/ユーザー | $1.98/月（粗利率99%） |
| 目標達成必要有料ユーザー | 10人 |
| 想定月間チャーン | 5% |
| LTV | $40（平均20ヶ月） |
| LTV/CAC | ∞ |
