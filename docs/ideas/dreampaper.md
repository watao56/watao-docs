# 🖼️ DreamPaper — AI壁紙ジェネレーター＆デイリー配信

## 概要

AIで自分好みの壁紙を無限に生成できるWebアプリ＆モバイル対応サービス。気分・季節・天気・好きなテーマに連動して毎日新しい壁紙を自動生成＆配信。iPhone/Android/PC全対応の解像度で出力。美しいギャラリーで他ユーザーの作品も閲覧・ダウンロード可能。

## 海外事例分析

| サービス | 規模 | 特徴 |
|---------|------|------|
| **Midjourney** | $200M+ ARR | AI画像生成の巨人。壁紙特化ではない |
| **Patterned.ai** | $5K+ MRR | AIパターン生成。壁紙にも使える |
| **AI Wallpaper apps (iOS)** | 多数 | App Storeで上位。$4.99-9.99/週の高単価 |
| **Unsplash** | 買収済 | 無料写真だがAI生成なし |
| **FLAVOR (日本)** | — | AIイラスト生成だが壁紙特化ではない |

**日本市場の機会**: スマホ壁紙は「毎日変えたい」層が一定数いる。日本のアニメ/和風/季節感を活かしたAI壁紙は海外アプリにない切り口。iOSのショートカットで自動壁紙変更と連携すれば習慣化しやすい。

## ターゲット

### プライマリ
- **スマホカスタマイズ好き（10-30代）**: 壁紙を頻繁に変える層
- **デスクトップ美化勢**: r/unixporn, r/desktops的なこだわり層
- **クリエイター**: 素材として使いたい

### セカンダリ
- カフェ・ショップ（デジタルサイネージ用）
- ブロガー・YouTuber（サムネ背景素材）

## 料金

| プラン | 月額 | 内容 |
|-------|------|------|
| **Free** | ¥0 | 日3枚生成、SD画質（1080p）、透かし小 |
| **Plus** | ¥480/月 | 日30枚、4K/8K、透かしなし、デイリー自動配信、全スタイル |
| **Pro** | ¥1,480/月 | 無制限、API、商用利用OK、カスタムスタイル学習 |

## ユーザーフロー

```
1. サインアップ（Google/Apple）
2. 好みの設定（スタイル: アニメ/写実/抽象/和風/ミニマル/サイバー）
3. テーマ入力 or おまかせ
4. 「生成」→ 10秒で壁紙表示
5. 気に入ったら「ダウンロード」（デバイス解像度自動判定）
6. 「デイリー配信ON」→ 毎朝通知で新壁紙が届く
7. ギャラリーで他ユーザー作品を閲覧・いいね・DL
```

## デザインコンセプト

- **"毎日のワクワク"**: アプリを開くたびに新しい驚き
- Pinterestライクなマソンリーギャラリー
- 壁紙プレビューがリアルなiPhone/Macモックアップ上で表示
- カテゴリ（季節/時間帯/色合い/ムード）でブラウズ
- 生成中のプログレスアニメーション（ノイズから徐々に画像が浮かぶ）

## アーキテクチャ

```
[Webアプリ (Next.js/PWA)] → [API (Next.js API Routes)]
                                    ↓
                          [生成キュー (Upstash Redis)]
                                    ↓
                          [Replicate API (SDXL/Flux)]
                                    ↓
                          [画像処理 (Sharp: アップスケール/フォーマット)]
                                    ↓
                          [S3 + CloudFront CDN]
                                    ↓
                          [デイリー配信 (Cron → Push/Email)]
```

## DB設計

```sql
users (
  id UUID PK,
  email TEXT UNIQUE,
  plan TEXT DEFAULT 'free',
  preferences JSONB, -- {styles: [...], themes: [...], device: "iphone15"}
  daily_enabled BOOLEAN DEFAULT false,
  generation_count_today INT DEFAULT 0,
  stripe_customer_id TEXT,
  created_at TIMESTAMPTZ
)

wallpapers (
  id UUID PK,
  user_id UUID FK → users,
  prompt TEXT,
  style TEXT,
  image_url TEXT,
  thumbnail_url TEXT,
  width INT,
  height INT,
  is_public BOOLEAN DEFAULT true,
  likes_count INT DEFAULT 0,
  downloads_count INT DEFAULT 0,
  created_at TIMESTAMPTZ
)

collections (
  id UUID PK,
  user_id UUID FK → users,
  name TEXT,
  wallpaper_ids UUID[],
  created_at TIMESTAMPTZ
)

daily_deliveries (
  id UUID PK,
  user_id UUID FK → users,
  wallpaper_id UUID FK → wallpapers,
  delivered_at TIMESTAMPTZ
)
```

## コスト見積もり

| 項目 | 月額（100ユーザー時） |
|------|----------------------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| Replicate API（SDXL/Flux） | ~$8（4000枚/月: $0.002/枚） |
| S3 + CloudFront | ~$2 |
| Upstash Redis (Free) | $0 |
| ドメイン | ~$1 |
| **合計** | **~$11/月** |

## MVPスコープ（2週間）

### Week 1
- [ ] Auth（Google OAuth）
- [ ] プロンプト入力 → Flux生成
- [ ] スタイル5種
- [ ] 解像度自動判定＆ダウンロード
- [ ] ギャラリー（公開壁紙一覧）

### Week 2
- [ ] デイリー配信（Email/PWA Push）
- [ ] いいね＆コレクション
- [ ] Stripe決済
- [ ] PWA対応（モバイルホーム画面追加）
- [ ] LP

## マーケ計画

### フェーズ1（ローンチ〜1ヶ月）
- Twitter/X で美麗壁紙を毎日投稿（「今日のAI壁紙」シリーズ）
- Reddit r/wallpapers, r/iOSsetups に投稿
- Product Hunt ローンチ

### フェーズ2（2-3ヶ月）
- Instagram/Pinterest でビジュアル訴求
- SEO:「AI 壁紙 生成」「iPhone 壁紙 おしゃれ」
- Apple Shortcutsとの連携方法をブログ記事化

### フェーズ3（3-6ヶ月）
- iOS/Androidアプリ化（React Native）
- クリエイターマーケットプレイス
- 企業向けデジタルサイネージプラン

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フレームワーク | Next.js 14 (PWA) |
| スタイリング | Tailwind CSS + shadcn/ui |
| 画像生成 | Replicate (Flux / SDXL) |
| 画像処理 | Sharp |
| キュー | Upstash Redis |
| DB/Auth | Supabase |
| Storage/CDN | S3 + CloudFront |
| 決済 | Stripe |
| Push通知 | Web Push API |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| AI生成コストが売上超過 | 赤字 | Free制限厳格化、キャッシュ（類似プロンプト） |
| 品質のムラ | 解約 | ネガティブプロンプト自動付加、品質フィルタ |
| ストレージ肥大 | コスト増 | 90日古いFree壁紙を自動削除、WebP圧縮 |
| 著作権/NSFW | 法的リスク | NSFWフィルタ、利用規約 |

## 競合分析

| 競合 | 強み | 弱み | DreamPaperの優位性 |
|------|------|------|-------------------|
| Midjourney | 最高品質 | $10/月〜、壁紙特化でない | 壁紙に最適化、¥480/月 |
| iOS壁紙アプリ | ストア内完結 | $4.99/週と高額 | 1/4の価格、Web完結 |
| Unsplash | 無料 | AI生成なし、カスタム不可 | 完全カスタム |
| 壁紙配布サイト | 大量素材 | 同じ画像の使い回し | 毎日ユニーク |

## $20達成シナリオ

```
目標: $20/月 ≒ ¥3,000/月

シナリオ: Plus 6人
- Plus ¥480 × 6 = ¥2,880 ≒ $20
- 必要Free登録: ~150人（Plus転換率4%）
- 達成時期: ローンチ後2-3ヶ月

楽観シナリオ:
- Plus 10人 + Pro 2人 = ¥7,760 ≒ $52
- ローンチ後1-2ヶ月（SNSバズ効果）
```

## ユニットエコノミクス

```
ARPU（Plus）: ¥480/月
AI生成コスト/ユーザー: ~¥40/月（日15枚 × 30日 × ¥0.09）
インフラ費/ユーザー: ~¥10/月
粗利: ¥430/月 (89.6%)
Stripe手数料: 3.6% = ¥17
純利: ¥413/ユーザー/月

LTV（想定6ヶ月）: ¥2,478
CAC目標: <¥800
LTV/CAC: >3x ✅

損益分岐: Plus 4人で黒字（インフラ~$11/月 at 100 users）
注意: AI生成コストがスケールと共に線形増加するため、
      Freeユーザーの生成制限がカギ
```
