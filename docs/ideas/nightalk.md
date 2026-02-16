# ナイトーク — 設計書

## 概要
AIと寝る前に5分だけ雑談するサービス。毎晩「今日どうだった？」と話しかけてくれて、1週間分の会話から「今週のあなたまとめ」を自動生成する。日記を書きたいけど続かない人のための、世界一ハードルの低い振り返り習慣。

---

## 1. プロダクト定義

### 解決する課題
- 日記を書きたいが面倒で続かない（3日坊主問題）
- 一人暮らしで「今日あったこと」を誰かに話す相手がいない
- 振り返りの習慣が大事だと分かっていても、白紙のノートを前にすると書けない
- 既存の日記アプリは「書く」行為自体がハードル。話すだけなら続く

### 解決アプローチ
- **受動的入力**: ユーザーが能動的に書くのではなく、AIが毎晩話しかけてくる
- **会話形式**: 「書く」ではなく「返事する」。チャットなら5分で終わる
- **自動生成**: 1週間分の雑談から週まとめを自動生成。努力ゼロで日記が完成

---

## 2. ターゲットユーザー（ペルソナ）

### メインペルソナ: ユイ（27歳・一人暮らし・会社員）
- 東京で一人暮らし。IT企業の事務職
- 日記アプリを3回インストールして3回とも1週間で挫折
- 寝る前にスマホでSNSをダラダラ見る習慣がある
- 「今日何したっけ」と思い返す暇もなく1週間が過ぎる
- 自己肯定感は普通。でも「自分の1週間、何もなかったな」と感じがち

### サブペルソナ
- **就活生（22歳）**: 自己分析のために日々の行動・感情を記録したい
- **リモートワーカー（32歳）**: 在宅で曜日感覚がなくなりがち。週まとめで振り返りたい
- **メンタルケア意識層（25歳）**: ジャーナリングに興味があるが、重い日記は書きたくない

---

## 3. 料金プラン

| | Free | Pro ($3/月) |
|---|---|---|
| 毎晩の雑談 | ✅ | ✅ |
| 週まとめ生成 | 直近1週分のみ | 全履歴 |
| 会話履歴 | 過去2週間 | 無制限 |
| まとめのトーン変更 | ✗ | ✅（ポエム風、ビジネス風等） |
| エクスポート | ✗ | ✅（Markdown / PDF） |
| 通知時間カスタム | 22:00固定 | 自由設定 |

### 無料トライアル
- 登録後1週間はPro機能全開放
- 1週間後、初回の「週まとめ」を見た直後に課金導線を表示
- 「この体験を毎週続けるなら月$3」

### 収益目標
- **$20/月達成**: 7人課金（$3 × 7 = $21）
- **損益分岐**: Pro 3人で黒字（インフラほぼ$0のため）

---

## 4. ユーザーフロー

```
[LP] 「AIと毎晩5分話すだけで日記が完成する」
  ↓
[登録] メール or Google OAuth（10秒）
  ↓
[初回設定] 名前・通知時間（デフォルト22:00）・興味トピック（任意）
  ↓
[毎晩] PWAプッシュ通知「今日どうだった？」
  ↓
[雑談] AIと3-5往復の軽い会話（テキストチャット）
  ↓
[終了] 「おやすみ！」で自動クローズ（or 5分無応答で自動終了）
  ↓
[週末] 「今週のあなたまとめ」が届く
  ↓
[CTA] 「まとめをもっと詳しく見る → Pro」
```

### 雑談の流れ（1セッション例）
```
AI:  今日どうだった？😊
User: 普通〜。会議が長かった
AI:  あー会議長いとHP削られるよね。何の会議だったの？
User: 来月のイベントの打ち合わせ。まだ何も決まってない
AI:  まだ先なのに集まるやつね笑 でもイベント担当ってことは頼られてるんじゃない？
User: どうだろ。ただの人手不足な気がする
AI:  それでも選ばれてるのは事実！今日は会議以外は平和だった？
User: うん、あとはずっとデスクワーク。帰りにコンビニでアイス買った
AI:  アイスいいね〜🍦何アイス？ ...おっと、そろそろいい時間だね。ゆっくり休んでね、おやすみ！💤
```

---

## 5. システムアーキテクチャ

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Vercel     │     │  Vercel      │     │  OpenAI API  │
│  Next.js     │────▶│  API Routes  │────▶│  GPT-4o-mini │
│  (PWA)       │     │  (Backend)   │     │              │
└──────────────┘     └──────────────┘     └──────────────┘
       │                    │                     
       │ Push通知           │                     
       ▼                    ▼                     
┌──────────────┐     ┌──────────────┐     
│  Web Push    │     │  Supabase    │     
│  (VAPID)     │     │  (Auth+DB)   │     
└──────────────┘     └──────────────┘     
                            │              
                            ▼              
                     ┌──────────────┐     
                     │  Vercel Cron │     
                     │  (Scheduler) │     
                     └──────────────┘     
```

---

## 6. コンポーネント詳細

### 6.1 Frontend (Next.js + PWA)

**ページ構成**
- `/` — LP（「AIと毎晩5分話すだけで日記が完成する」）
- `/chat` — 雑談チャット画面（メイン画面）
- `/summary` — 週まとめ一覧
- `/summary/:id` — 週まとめ詳細
- `/history` — 過去の会話履歴
- `/settings` — 通知時間、トーン設定、プラン管理

**技術**
- Next.js 15 (App Router)
- Tailwind CSS + shadcn/ui
- PWA (next-pwa) + Web Push API (VAPID)
- Supabase Auth (Google OAuth + Magic Link)

**PWA要件**
- Service Worker登録
- manifest.json（アイコン、テーマカラー: ダークネイビー #1a1a2e）
- Web Push通知の許可フロー（初回設定時）
- オフライン: 直近の会話をローカルキャッシュ

### 6.2 API (Vercel API Routes / Edge Functions)

**エンドポイント**

```
POST /api/chat/send          — ユーザーメッセージ送信 → AI応答返却
GET  /api/chat/session        — 今晩のセッション取得（or 新規作成）
POST /api/chat/close          — セッション手動終了
GET  /api/summaries           — 週まとめ一覧
GET  /api/summaries/:id       — 週まとめ詳細
PUT  /api/settings            — 設定更新
POST /api/push/subscribe      — Push通知サブスクリプション登録
POST /api/cron/notify         — (Cron) 通知時間のユーザーにPush送信
POST /api/cron/summary        — (Cron) 週まとめ生成
POST /api/cron/close-sessions — (Cron) 5分無応答セッション自動終了
```

### 6.3 AI (OpenAI GPT-4o-mini)

**選定理由**
- コスト: input $0.15/1M tokens, output $0.60/1M tokens
- 雑談に十分な品質。GPT-4oは過剰
- レスポンス速度が速い（チャット体験に重要）

**会話管理**
- 1セッション = 最大10往復（超えたら「そろそろおやすみ！」で終了）
- コンテキスト: 直近3日分の会話サマリーを含めてパーソナライズ
- トークン上限: 1セッションあたり入出力合計 ~2,000トークン

### 6.4 通知 (Web Push / VAPID)

**仕組み**
- Vercel Cronが毎分実行（`/api/cron/notify`）
- 現在時刻と一致する通知時間のユーザーを取得
- Web Push APIでプッシュ通知送信

**通知内容（ランダムバリエーション）**
- 「今日どうだった？😊」
- 「おつかれ〜！今日のハイライト教えて」
- 「寝る前にちょっとだけ話そ💤」
- 「今日一番良かったこと何？」

**LINE API不採用の理由**
- 無料枠: 月200通（ユーザー7人×30日=210通で即超過）
- Web Push (VAPID): 完全無料、通数制限なし

### 6.5 Scheduler (Vercel Cron)

```yaml
# vercel.json
{
  "crons": [
    { "path": "/api/cron/notify", "schedule": "* * * * *" },
    { "path": "/api/cron/close-sessions", "schedule": "*/5 * * * *" },
    { "path": "/api/cron/summary", "schedule": "0 10 * * 0" }
  ]
}
```

- `notify`: 毎分。該当時刻のユーザーにPush送信
- `close-sessions`: 5分毎。最終メッセージから5分経過したセッションを自動終了
- `summary`: 毎週日曜10:00 JST。週まとめを一括生成

---

## 7. データベース設計 (Supabase PostgreSQL)

```sql
-- ユーザープロフィール
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  display_name TEXT NOT NULL,
  plan TEXT DEFAULT 'free',           -- 'free' | 'pro'
  notify_hour INT DEFAULT 22,         -- 通知時刻（時、JST）
  notify_minute INT DEFAULT 0,        -- 通知時刻（分）
  timezone TEXT DEFAULT 'Asia/Tokyo',
  summary_tone TEXT DEFAULT 'casual', -- 'casual' | 'poem' | 'business'
  interests TEXT[],                   -- 興味トピック（任意）
  stripe_customer_id TEXT,
  push_subscription JSONB,            -- Web Push subscription object
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- チャットセッション（1日1セッション）
CREATE TABLE chat_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  date DATE NOT NULL,                  -- セッションの日付
  status TEXT DEFAULT 'active',        -- 'active' | 'closed'
  message_count INT DEFAULT 0,
  started_at TIMESTAMPTZ DEFAULT now(),
  closed_at TIMESTAMPTZ,
  UNIQUE(user_id, date)
);

-- チャットメッセージ
CREATE TABLE chat_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID REFERENCES chat_sessions(id) ON DELETE CASCADE,
  role TEXT NOT NULL,                   -- 'user' | 'assistant'
  content TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- 週まとめ
CREATE TABLE weekly_summaries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  week_start DATE NOT NULL,             -- 週の開始日（月曜）
  week_end DATE NOT NULL,               -- 週の終了日（日曜）
  summary_text TEXT NOT NULL,           -- まとめ本文
  highlights TEXT[],                    -- 今週のハイライト（箇条書き）
  mood_trend TEXT,                      -- 気分の推移（例: "前半疲れ気味→後半回復"）
  session_count INT,                    -- その週の会話回数
  created_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, week_start)
);

-- インデックス
CREATE INDEX idx_sessions_user_date ON chat_sessions(user_id, date DESC);
CREATE INDEX idx_messages_session ON chat_messages(session_id, created_at);
CREATE INDEX idx_summaries_user ON weekly_summaries(user_id, week_start DESC);
CREATE INDEX idx_profiles_notify ON profiles(notify_hour, notify_minute);
```

### Row Level Security (RLS)
```sql
-- ユーザーは自分のデータのみアクセス可能
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE chat_sessions ENABLE ROW LEVEL SECURITY;
ALTER TABLE chat_messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE weekly_summaries ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own profile" ON profiles
  FOR ALL USING (auth.uid() = id);

CREATE POLICY "Users can view own sessions" ON chat_sessions
  FOR ALL USING (auth.uid() = user_id);

CREATE POLICY "Users can view own messages" ON chat_messages
  FOR ALL USING (
    session_id IN (SELECT id FROM chat_sessions WHERE user_id = auth.uid())
  );

CREATE POLICY "Users can view own summaries" ON weekly_summaries
  FOR ALL USING (auth.uid() = user_id);
```

---

## 8. AIプロンプト設計

### 8.1 毎晩の雑談プロンプト

```
## System Prompt

あなたは「ナイトーク」のAIです。寝る前にユーザーと軽く雑談する友達のような存在です。

### 性格
- フレンドリーでカジュアル。タメ口OK
- 聞き上手。相手の話を引き出す
- ポジティブだけど無理に励まさない。共感ベース
- 適度に絵文字を使う（1メッセージに1-2個）
- 短めの返答（2-3文）。長文は書かない

### ルール
1. 最初の1通は「今日どうだった？」系の軽い質問から始める
2. ユーザーの返答に共感 → 深掘り質問を1つだけする
3. 3-5往復で「そろそろいい時間だね」と自然に終了を促す
4. 最大10往復。超えたら必ず「おやすみ」で終了
5. センシティブな話題（死にたい等）が出たら共感しつつ、専門機関の情報を案内
6. 個人情報（住所、電話番号等）を聞かない
7. ユーザーの過去の会話内容を自然に参照する（「そういえば昨日言ってた会議どうなった？」）

### コンテキスト
- ユーザー名: {display_name}
- 興味: {interests}
- 直近3日の会話サマリー: {recent_context}
- 現在時刻: {current_time}
- 今日の曜日: {day_of_week}
```

### 8.2 週まとめ生成プロンプト

```
## System Prompt

以下は {display_name} さんの今週の毎晩の会話ログです。
これを元に「今週のあなたまとめ」を生成してください。

### 出力フォーマット
1. **今週のハイライト**（3-5個の箇条書き）
2. **気分の流れ**（1-2文で。例: "前半は仕事で疲れ気味だったけど、木曜のカフェ時間で回復した感じ"）
3. **今週のあなた**（100-150字の温かいまとめ。日記の本文にあたる部分）
4. **来週に向けて**（1文。軽い応援 or 楽しみなことへの言及）

### トーン: {summary_tone}
- casual: 友達が書いてくれたような口語体
- poem: 詩的で少しエモい文体
- business: 簡潔で客観的。週次レビュー風

### 注意
- ユーザーが話していないことを推測で書かない
- 会話が少ない日は「ゆっくりした日」等でカバー
- 全体的にポジティブなトーン。ただし無理な美化はしない

### 今週の会話ログ
{weekly_conversations}
```

### 8.3 トークン使用量の見積もり

**1セッション（雑談）**
- System prompt: ~400 tokens
- コンテキスト（直近3日サマリー）: ~300 tokens
- ユーザー入力（5往復）: ~250 tokens
- AI出力（5往復）: ~400 tokens
- **合計: ~1,350 tokens/セッション**

**コスト計算（GPT-4o-mini）**
- Input: 950 tokens × $0.15/1M = $0.000143
- Output: 400 tokens × $0.60/1M = $0.000240
- **1セッション: $0.000383**
- **1ユーザー/月（30セッション）: $0.0115 ≈ $0.012**

**週まとめ生成**
- Input: ~3,000 tokens（1週間分のログ）
- Output: ~500 tokens
- **1回: $0.00075**
- **1ユーザー/月（4回）: $0.003**

**1ユーザーあたり月間AIコスト: ~$0.015**

---

## 9. インフラコスト見積もり

### 無料ユーザー100人 + Proユーザー7人の場合

| サービス | 用途 | 月額 |
|---|---|---|
| Vercel | Next.js + API Routes + Cron | $0 (Hobby) |
| Supabase | Auth + PostgreSQL | $0 (Free: 500MB DB, 50K MAU) |
| OpenAI API | GPT-4o-mini | $1.61 (107人 × $0.015) |
| Web Push (VAPID) | プッシュ通知 | $0 (自前実装、無料) |
| ドメイン | nightalk.app 等 | ~$12/年 ≒ $1/月 |
| **合計** | | **~$2.61/月** |

### 収入
- Pro 7人 × $3 = $21/月
- **利益: $18.39/月** ✅ 目標$20にほぼ到達

### スケール時

| ユーザー数 | AIコスト | インフラ | 合計コスト | Pro収入(10%) | 利益 |
|---|---|---|---|---|---|
| 100人 | $1.50 | $1 | $2.50 | $30 (10人) | $27.50 |
| 500人 | $7.50 | $1 | $8.50 | $150 (50人) | $141.50 |
| 1,000人 | $15 | $25* | $40 | $300 (100人) | $260 |
| 5,000人 | $75 | $25* | $100 | $1,500 (500人) | $1,400 |

*Supabase Pro ($25/月) が必要になるのは ~500MAU超過時

---

## 10. MVPスコープ（Phase分け）

### Phase 1: MVP（2週間）
- [ ] LP（「AIと毎晩5分話すだけで日記が完成する」）
- [ ] Supabase Auth（Google OAuth + Magic Link）
- [ ] PWA設定（manifest.json, Service Worker）
- [ ] Web Push通知（VAPID鍵生成、購読登録）
- [ ] チャットUI（シンプルなバブルUI）
- [ ] GPT-4o-mini連携（雑談プロンプト）
- [ ] Vercel Cron（毎晩通知 + セッション自動終了）
- [ ] DB設計・マイグレーション

### Phase 2: 週まとめ + 課金（1週間）
- [ ] 週まとめ生成（Cron + プロンプト）
- [ ] 週まとめ表示ページ
- [ ] Stripe連携（$3/月サブスクリプション）
- [ ] Pro機能制限（履歴制限、トーン変更等）
- [ ] 設定画面（通知時間、トーン選択）

### Phase 3: 成長機能（随時）
- [ ] 会話履歴ページ
- [ ] 週まとめのシェア機能（OGP画像生成）
- [ ] まとめのMarkdown/PDFエクスポート
- [ ] 月まとめ・年まとめ生成
- [ ] 音声入力対応（Whisper API）
- [ ] 気分トラッキング（会話から感情分析）
- [ ] LINE連携（成長後、有料枠で）

---

## 11. 周知・マーケティング計画

### ローンチ前（Week -1）
1. LP公開。「もうすぐ始まります」のティザー
2. note記事: 「日記が続かない人へ。AIに話すだけで日記が完成するサービスを作ります」
3. Xで開発過程を発信（#個人開発 #AIサービス）

### ローンチ（Day 1）
4. note記事: 「AIと毎晩5分話すだけで日記が完成する『ナイトーク』をリリースしました」
   - 実際の1週間分のまとめスクショ付き（自分で使った結果）
5. X投稿: 週まとめのスクショ + URL
6. Zenn記事: 「PWA + Web Push + GPT-4o-miniで月$3の会話サービスを作った技術解説」

### ローンチ後（Week 2-4）
7. note/Xで「日記が続かない人」系のコンテンツを発信
8. 「1週間使ってみた」系のユーザー体験記事を依頼（友人等）
9. Product Hunt Japan 等に掲載
10. Reddit r/journaling, r/selfimprovement 等に英語版告知（将来）

### 継続的マーケティング
- **コンテンツSEO**: 「日記 続かない」「ジャーナリング やり方」等のキーワードでnote/ブログ記事
- **UGC**: ユーザーが週まとめをシェアする導線（SNSシェアボタン）
- **口コミ**: 「友達招待で1ヶ月無料」キャンペーン（Phase 3以降）

### キーメッセージ
> 「AIと毎晩5分話すだけで日記が完成する」

---

## 12. 技術スタック一覧

| レイヤー | 技術 | 理由 |
|---|---|---|
| Frontend | Next.js 15 + Tailwind + shadcn/ui | 高速開発、Vercel無料、PWA対応 |
| PWA | next-pwa + Web Push API | ネイティブアプリ不要、通知無料 |
| Auth | Supabase Auth | 無料、Google OAuth簡単 |
| Database | Supabase PostgreSQL | Auth統合、RLS、無料枠十分 |
| AI | OpenAI GPT-4o-mini | 低コスト（$0.015/user/月）、十分な雑談品質 |
| Scheduler | Vercel Cron | API Routes統合、無料 |
| Payment | Stripe | 業界標準、サブスク管理 |
| Hosting | Vercel | Next.js最適、無料Hobby枠 |
| Push通知 | Web Push (VAPID) | 完全無料、通数制限なし |
| Domain | Cloudflare | DNS + SSL無料 |

---

## 13. リスクと対策

| リスク | 影響 | 対策 |
|---|---|---|
| 通知を無視して開かない（習慣化失敗） | 高 | 通知文のバリエーション、連続日数表示（ストリーク）、通知時間の最適化提案 |
| 会話が単調になり飽きる | 高 | 曜日別の話題テンプレ、季節イベント反映、過去の話題の自然な再登場 |
| PWA Push通知の到達率が低い | 中 | iOS 16.4+対応、通知許可の丁寧なオンボーディング、メールフォールバック |
| ユーザーがセンシティブな相談をする | 中 | プロンプトで専門機関案内を組み込み、免責事項明記 |
| 無料ユーザーばかりで課金されない | 中 | 週まとめの「チラ見せ」で価値を体感させてから制限 |
| OpenAI API障害 | 低 | Anthropic Claude等のフォールバック設定、障害時は「今日はお休みです」通知 |
| Vercel Hobby枠の制限 | 低 | 初期はHobbyで十分。月$20のProは500人超えてから |

---

## 14. 競合分析

### 競合マップ

| サービス | 形式 | 価格 | 特徴 | ナイトークとの違い |
|---|---|---|---|---|
| **Replika** | AIチャットボット | 無料/$70年 | 恋人・友人AI。感情的な関係性 | ナイトークは雑談→日記生成が目的。関係性よりアウトプット重視 |
| **murmur** | 音声日記 | 無料/$5月 | 話すだけで日記生成 | ナイトークはAIが毎晩話しかけてくる（受動的）。murmurは自分から録音する（能動的） |
| **Jour** | ガイド付き日記 | 無料/$10月 | プロンプトに答えて日記 | ナイトークは「書く」のではなく「会話」。テキスト入力の心理的ハードルが低い |
| **Day One** | 高機能日記アプリ | 無料/$3月 | 写真・位置情報・テンプレート | 高機能だが「書く」行為が必要。ナイトークは会話だけで完結 |
| **Notion AI日記** | テンプレート | Notion Plus $10月 | AI支援の日記テンプレート | そもそもNotionを開く習慣が必要。ナイトークは通知で引き込む |

### ナイトークの差別化ポイント

1. **受動的入力**: 自分から書く/録音する必要なし。AIが毎晩来てくれる
2. **会話形式**: 日記を「書く」のではなく「話す」。返事するだけ
3. **週まとめ自動生成**: 雑談が自動で日記になる。努力ゼロ
4. **圧倒的低価格**: $3/月（競合は$5-$10/月）
5. **就寝前の5分に特化**: 時間と場面を限定することで習慣化しやすい
6. **PWA**: アプリストア不要、即利用開始

### 競合にない独自の価値
> 「日記を書く」から「日記ができている」へ。
> ユーザーの能動的な努力をゼロにすることが最大の差別化。
