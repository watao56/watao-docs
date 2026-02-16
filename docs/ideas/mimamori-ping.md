# 👴 MimamoriPing — 離れて暮らす親の安否確認サービス

> **「なぜ金を払うか」**: 離れて暮らす高齢の親が元気かどうか、毎日自動で確認して通知してくれるため。

## 1. 概要・解決する課題

高齢の親と離れて暮らす子世代にとって、「親が今日も元気か」は毎日の不安。かといって毎日電話するのは互いに負担。

MimamoriPingは**毎日決まった時間にLINEで「元気ですか？」と自動送信。親が「元気」ボタンを押すだけ。押さなかったら家族に通知**。

- 親: ボタン1つ押すだけ。スマホ操作のハードルゼロ
- 子: 毎日「今日も返事があった」で安心。返事がなければすぐ確認できる

### 市場の痛み
- 一人暮らし高齢者: 約700万人（総務省統計）
- 「毎日親のことが心配」な40-50代: 離れて暮らす子世代の78%
- 既存の見守りサービス: 月額3,000-5,000円で高い。カメラやセンサーは大掛かり

## 2. ターゲットユーザー（具体的ペルソナ）

### ペルソナA: 会社員 ケイコ（48歳）
- 東京在住。実家の母（76歳）は地方で一人暮らし
- 毎日心配だが、毎日電話すると母が「しつこい」と嫌がる
- 「元気の確認だけでいい。異変があれば知りたい」

### ペルソナB: 共働き夫婦 タロウ（42歳）
- 妻の父（80歳）が一人暮らし。認知症の初期症状あり
- 高額な見守りセンサーを検討中だが、導入が大変
- 「まずは毎日の安否確認だけでいい」

### ペルソナC: 転勤族 アキ（38歳）
- 両親（70代）と離れて暮らす。転勤先から毎週電話
- 「電話の間に何かあったら」という不安
- シンプルで安いサービスがほしい

## 3. 料金プラン

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 1人見守り、メール通知のみ、応答なし通知は翌日 |
| Pro | $3/月 ($30/年) | 3人まで見守り、LINE/メール通知、応答なし30分で通知、応答履歴カレンダー |

## 4. ユーザーフロー

### 子世代（設定する側）
```
1. メールでサインアップ
2. 見守り対象を登録（親の名前、LINE ID or メール）
3. 確認メッセージの送信時間を設定（例: 毎朝9:00）
4. 応答がない場合の通知先を設定（自分+兄弟等）
5. ダッシュボードで応答履歴を確認（カレンダー表示）
```

### 親世代（見守られる側）
```
1. LINEで「MimamoriPing」を友だち追加
2. 毎朝9:00にメッセージ受信
   「おはようございます☀️ 今日もお元気ですか？
    元気なら下のボタンを押してくださいね！」
   [😊 元気です] [😴 まあまあ] 
3. ボタンを押すだけ（テキスト入力不要）
4. 押さなかった場合、1時間後にリマインド
5. リマインドにも応答なし → 家族に通知
```

## 5. システムアーキテクチャ

```
[Vercel Cron] → [API Routes] → [LINE Messaging API]
     (毎朝9:00)        ↓              ↓
                 [Supabase]    [見守り対象にメッセージ送信]
                      ↓
              [応答チェック] → 未応答 → [家族にアラート通知]
                                         (LINE/メール)
```

## 6. コンポーネント詳細

### フロントエンド (Next.js)
- `/` — LP（感情に訴えるコピー）
- `/dashboard` — 応答履歴カレンダー（✅/❌/😴表示）
- `/settings` — 見守り対象設定、通知先設定、送信時間設定

### バックエンド
- `POST /api/line/webhook` — LINEからの応答受信
- `POST /api/check/morning` — Cronから呼ばれ、確認メッセージ送信
- `POST /api/check/remind` — 未応答者へリマインド送信（1時間後）
- `POST /api/check/alert` — リマインドも未応答 → 家族に通知
- `GET /api/history` — 応答履歴取得

### Cronスケジュール
- 09:00 JST: 確認メッセージ送信
- 10:00 JST: 未応答者にリマインド
- 10:30 JST: リマインドも未応答 → 家族にアラート

## 7. データベース設計

```sql
CREATE TABLE families (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id UUID REFERENCES auth.users(id) ON DELETE CASCADE, -- 子世代
  plan TEXT DEFAULT 'free',
  stripe_customer_id TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE watch_targets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  family_id UUID REFERENCES families(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  line_user_id TEXT, -- LINE連携時
  check_time TIME DEFAULT '09:00',
  timezone TEXT DEFAULT 'Asia/Tokyo',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE alert_contacts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  family_id UUID REFERENCES families(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  contact_type TEXT NOT NULL, -- 'line' | 'email'
  contact_value TEXT NOT NULL, -- LINE user ID or email
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE check_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  target_id UUID REFERENCES watch_targets(id) ON DELETE CASCADE,
  check_date DATE NOT NULL,
  message_sent_at TIMESTAMPTZ,
  response_at TIMESTAMPTZ,
  response_status TEXT, -- 'genki' | 'maamaa' | 'no_response'
  remind_sent_at TIMESTAMPTZ,
  alert_sent_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(target_id, check_date)
);

CREATE INDEX idx_check_logs_target_date ON check_logs(target_id, check_date);
CREATE INDEX idx_check_logs_status ON check_logs(response_status) WHERE response_status = 'no_response';
```

## 8. インフラ+AIコスト見積もり

| 項目 | 月額コスト | 根拠 |
|------|-----------|------|
| Vercel (Hobby) | $0 | 無料枠。Cron 1回/日→3回/日は要Proだが、1つのCronで3段階処理可能 |
| Supabase (Free) | $0 | 無料枠 |
| LINE Messaging API | $0-$5 | 無料枠200通/月。超過時はライトプラン月5,000円（15,000通） |
| SendGrid (Free) | $0 | アラート通知用 |
| ドメイン | $1/月 | |
| **合計** | **$1-6/月** | |

### LINE通知コスト詳細
- 1人の見守り対象: 月30通（確認）+ 数通（リマインド）≈ 月35通
- 無料枠200通: 5-6人まで無料
- 有料ユーザー10人（各1人見守り）= 350通/月 → LINEライトプラン必要（月5,000円≈$33）
  - **これは高い。対策が必要**
  
### LINE通知コスト対策
- **Phase 1ではメール通知メインに変更**
- 親世代にはメール or Webページでボタンクリックでも可
- LINEは任意オプション。LINE通知はPro限定
- 10人までは無料枠内で運用可能

**修正後コスト**: $1/月（メール通知メイン時）

**AIコスト: $0**（AI不使用）

## 9. MVPスコープ

### Phase 1: MVP（2週間）
- ユーザー認証（子世代）
- 見守り対象登録
- 毎朝メール送信（確認ボタン付きメール）
- ボタンクリックで応答記録
- 未応答時のアラートメール（家族向け）
- 応答履歴ダッシュボード
- LP

### Phase 2: LINE対応+有料化（1週間）
- LINE Bot連携（見守り対象がLINEで応答）
- Stripe決済
- Pro機能（3人まで、LINE通知、30分アラート）

### Phase 3: 成長（1週間）
- 応答カレンダー（月表示、パターン分析）
- 「最近"まあまあ"が続いています」の気づき通知
- 家族グループ共有

## 10. 周知・マーケティング計画

### Twitter/X投稿例

```
👴 離れて暮らす親御さん、今日も元気ですか？

毎日電話するのは大変。
でも何かあったら…と不安。

毎朝自動で「元気？」と聞いて
返事がなかったら教えてくれるサービス作りました。

親はボタン1つ押すだけ。
子は安心が毎日届く。

無料で使えます👇
https://mimamoriping.com
```

```
実家の母が倒れて3日間誰も気づかなかった、
という知人の話を聞いて作りました。

毎朝「元気ですか？」とメッセージ。
返事がなかったら家族に通知。

たったそれだけ。
でもこの「たったそれだけ」がなかった。

MimamoriPing、無料で始められます👇
```

**投稿タイミング**: 敬老の日（9月）、お盆前後、年末年始（帰省シーズン）

### ターゲットコミュニティ
- 「介護」「親の見守り」系Facebookグループ
- 「離れて暮らす親」系コミュニティ
- r/japanlife
- note（「一人暮らしの親をどう見守るか」記事）
- Yahoo!知恵袋「親 見守り 安い」回答にリンク
- 地域包括支援センターへのアプローチ

### SEO記事
- 「高齢者 見守り 安い」
- 「一人暮らし 親 安否確認」
- 「見守りサービス 比較」

## 11. 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js 14, Tailwind CSS, shadcn/ui |
| バックエンド | Next.js API Routes |
| データベース | Supabase (PostgreSQL) |
| 認証 | Supabase Auth |
| メール通知 | SendGrid |
| LINE通知 | LINE Messaging API（Phase 2） |
| 決済 | Stripe |
| ホスティング | Vercel |
| Cron | Vercel Cron Jobs |

## 12. リスクと対策

| リスク | 影響 | 対策 |
|--------|------|------|
| 親がメール/LINEを使えない | 利用不可 | メール版はリンククリックのみ。LINEはボタンタップのみ。最低限の操作 |
| 毎日の通知がうざがられる | 親が応答しなくなる | 時間帯をカスタマイズ。メッセージのバリエーション（天気、季節の挨拶を添える） |
| 誤報（寝坊で応答遅れ） | 家族が不必要に心配 | リマインド→1.5時間待機→アラートの3段階。週末は送信時間を遅めに設定可 |
| LINE通知コストが高い | 赤字 | Phase 1はメール主体。LINEはPro限定で段階的に導入 |
| 大手参入（Apple Watch見守り等） | ユーザー流出 | Apple Watchは高価（5万円+）。月$3のシンプルさで棲み分け |
| 法的リスク（安否確認の責任） | 訴訟リスク | 利用規約で「緊急通報の代替ではない」を明記。免責事項を明確化 |

## 13. 競合分析・差別化

| 競合 | 特徴 | MimamoriPingの勝ち筋 |
|------|------|----------------------|
| ALSOK見守り（月3,300円） | センサー+駆けつけ | 月$3で1/10のコスト。「まず安否確認だけ」のライトニーズ |
| Apple Watch転倒検出 | 高機能だが5万円+ | 初期費用ゼロ。既存のスマホだけで利用可 |
| みまもりほっとライン（象印） | ポット使用で安否確認 | ポット不要。LINE/メールで直接的なコミュニケーション |
| LINE「みまもりBot」類似 | 個人開発の類似Bot | UI/UX、応答履歴カレンダー、多段階アラートで差別化 |
| まごチャンネル | 写真共有型見守り | 安否確認特化。シンプルで分かりやすい |

## 14. $20/月達成の現実的シナリオ

| 月 | 無料ユーザー | 有料ユーザー | MRR |
|----|-------------|-------------|-----|
| 1 | 30 | 3 | $9 |
| 2 | 70 | 8 | $24 ✅ |

**根拠**:
- 「親の見守り」は感情的に強いテーマ。SNS投稿の共感・拡散率が高い
- Free→Pro転換率10-15%（安否確認は「安心」に直結。ケチりにくい）
- 敬老の日（9月）、帰省シーズン投稿でバイラル効果
- 口コミ効果：「うちの親にも使わせたい」が自然に発生
- $3/月はランチ1回分。心理的ハードルが低い

## 15. ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| ARPU | $3/月 |
| CAC | $0 |
| インフラコスト/ユーザー | $0.05/月 |
| 粗利/ユーザー | $2.95/月（粗利率98.3%） |
| 目標達成必要有料ユーザー | 7人 |
| 想定月間チャーン | 3%（親が元気な限り解約しない） |
| LTV | $100（平均33ヶ月利用） |
| LTV/CAC | ∞ |
