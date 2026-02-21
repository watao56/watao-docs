# 🎨 MojiForge — AI絵文字/スタンプパック生成ツール

## 概要

テキストプロンプトやラフスケッチから、オリジナルの絵文字・スタンプパックをAI生成するWebアプリ。Slack/Discord/LINE用にフォーマット自動変換＆ダウンロード。チーム・コミュニティの一体感を「カスタム絵文字」で演出。個人クリエイターの「スタンプ販売」用途にも。

## 海外事例分析

| サービス | 規模 | 特徴 |
|---------|------|------|
| **Emoji Kitchen (Google)** | 10億+ ユーザー | 絵文字の組み合わせ。超人気だがカスタム生成ではない |
| **Fazier Emoji Maker** | — | AIで絵文字生成。Product Huntで話題 |
| **Bitmoji (Snap)** | 3億+ DL | アバター絵文字。カスタムスタンプではない |
| **LINE Creators Market** | ¥76億/年 | 日本最大のスタンプ販売プラットフォーム |

**日本市場の機会**: LINE Creators Marketは巨大だが、スタンプ制作には絵のスキルが必要。「絵が描けなくてもAIでプロ品質のスタンプパックが作れる」は強力な訴求。企業のSlack/Discord用カスタム絵文字需要も急増中。

## ターゲット

### プライマリ
- **Slack/Discordコミュニティ運営者**: オリジナル絵文字でコミュニティブランディング
- **LINE スタンプクリエイター志望者**: 絵が描けなくてもスタンプ販売したい層
- **中小企業マーケ担当**: 社内Slack用のオリジナル絵文字

### セカンダリ
- VTuber・配信者（ファン用スタンプ）
- 同人・二次創作コミュニティ

## 料金

| プラン | 月額 | 内容 |
|-------|------|------|
| **Free** | ¥0 | 月5パック生成、透かし付き、128px |
| **Creator** | ¥780/月 | 月50パック、透かしなし、高解像度、LINE/Slack/Discord形式出力 |
| **Team** | ¥1,980/月 | 無制限、チーム共有、ブランドガイドライン設定、API |

## ユーザーフロー

```
1. サインアップ
2. 「新しいパック作成」をクリック
3. スタイル選択（フラット/ピクセル/水彩/アニメ/3D/カワイイ）
4. プロンプト入力（例:「柴犬が色んな感情を表現」）
5. AI が 8-40個の絵文字/スタンプ候補を生成（30秒〜）
6. 気に入ったものを選択・微調整（色変更、テキスト追加）
7. 出力形式選択（Slack/Discord/LINE/PNG/GIF）
8. 一括ダウンロード or ワンクリックSlackインポート
```

## デザインコンセプト

- **"作る楽しさ"重視**: 生成過程をアニメーションで見せる
- ギャラリー機能で他ユーザーの作品を閲覧＆いいね
- Figma風のシンプルなエディタUI
- 生成結果のプレビューがチャットアプリ風モックアップ
- ダーク/ライトテーマ

## アーキテクチャ

```
[Webアプリ (Next.js)] → [API (Next.js API Routes)]
                              ↓
                    [画像生成キュー (Bull + Redis)]
                              ↓
                    [SDXL / DALL-E 3 API]
                              ↓
                    [後処理 (Sharp: リサイズ/背景除去/フォーマット変換)]
                              ↓
                    [S3 Storage] → [CloudFront CDN]
```

### コンポーネント
- **フロントエンド**: Next.js 14 + Tailwind CSS
- **画像生成**: Replicate API (SDXL Turbo) — 1枚 ~$0.002
- **後処理**: Sharp (Node.js、ローカル)
- **背景除去**: rembg (Python、Serverless)
- **キュー**: Upstash Redis (Free tier)
- **Storage**: S3 + CloudFront
- **DB**: Supabase
- **決済**: Stripe

## DB設計

```sql
users (
  id UUID PK,
  email TEXT UNIQUE,
  plan TEXT DEFAULT 'free',
  generation_count INT DEFAULT 0, -- 月間生成カウント
  stripe_customer_id TEXT,
  created_at TIMESTAMPTZ
)

packs (
  id UUID PK,
  user_id UUID FK → users,
  name TEXT,
  style TEXT, -- flat/pixel/watercolor/anime/3d/kawaii
  prompt TEXT,
  is_public BOOLEAN DEFAULT false,
  likes_count INT DEFAULT 0,
  created_at TIMESTAMPTZ
)

emojis (
  id UUID PK,
  pack_id UUID FK → packs,
  image_url TEXT,
  thumbnail_url TEXT,
  emotion TEXT, -- happy/sad/angry/etc
  text_label TEXT,
  format TEXT, -- png/gif/apng
  width INT,
  height INT,
  created_at TIMESTAMPTZ
)

exports (
  id UUID PK,
  pack_id UUID FK → packs,
  platform TEXT, -- slack/discord/line/raw
  file_url TEXT,
  created_at TIMESTAMPTZ
)
```

## コスト見積もり

| 項目 | 月額（50ユーザー時） |
|------|---------------------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| Replicate API（SDXL） | ~$3（1500枚/月想定） |
| Upstash Redis (Free) | $0 |
| S3 + CloudFront | ~$0.50 |
| ドメイン | ~$1 |
| **合計** | **~$4.50/月** |

## MVPスコープ（2週間）

### Week 1
- [ ] Auth（Google OAuth）
- [ ] プロンプト入力 → SDXL生成（8枚パック）
- [ ] スタイル3種（フラット/アニメ/ピクセル）
- [ ] 背景除去＆リサイズ

### Week 2
- [ ] Slack/Discord形式エクスポート
- [ ] ギャラリー（公開パック一覧）
- [ ] Stripe決済
- [ ] LP

## マーケ計画

### フェーズ1（ローンチ〜1ヶ月）
- Twitter/X で生成デモ動画（バズ狙い）
- Product Hunt ローンチ
- Discord/Slackコミュニティで「カスタム絵文字作ります」PR
- 「AIでLINEスタンプ作ってみた」系note記事

### フェーズ2（2-3ヶ月）
- LINEスタンプ副業系YouTuber/ブロガーにリーチ
- SEO:「オリジナル絵文字 作成」「LINEスタンプ AI」
- ギャラリーのSEO効果（UGC）

### フェーズ3（3-6ヶ月）
- LINE Creators Market連携（直接申請フロー）
- Figmaプラグイン
- APIプラン展開

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フレームワーク | Next.js 14 |
| スタイリング | Tailwind CSS + shadcn/ui |
| 画像生成 | Replicate (SDXL Turbo) |
| 画像処理 | Sharp + rembg |
| キュー | Upstash Redis + Bull |
| DB/Auth | Supabase |
| Storage/CDN | S3 + CloudFront |
| 決済 | Stripe |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| AI生成品質のブレ | ユーザー離脱 | スタイル別fine-tunedモデル、リトライ機能 |
| 著作権問題 | 法的リスク | 利用規約明記、商用利用可能モデル使用 |
| API費用スパイク | 赤字 | 月間生成上限、キャッシュ活用 |
| LINEスタンプ審査落ち | 不満 | ガイドライン準拠チェック機能 |

## 競合分析

| 競合 | 強み | 弱み | MojiForgeの優位性 |
|------|------|------|-------------------|
| 手描き制作 | 独自性 | スキル＆時間必要 | 30秒で生成 |
| Canva | 汎用デザイン | 絵文字特化ではない | 絵文字/スタンプに最適化 |
| AI画像生成(汎用) | 高品質 | フォーマット変換が手動 | Slack/Discord/LINE一括出力 |

## $20達成シナリオ

```
目標: $20/月 ≒ ¥3,000/月

シナリオ: Creator 4人
- Creator ¥780 × 4 = ¥3,120 ≒ $21
- 必要Free登録: ~80人（Creator転換率5%）
- 達成時期: ローンチ後2-3ヶ月

楽観シナリオ:
- Creator 6人 + Team 1人 = ¥6,660 ≒ $44
- ローンチ後1-2ヶ月（バズ効果）
```

## ユニットエコノミクス

```
ARPU（Creator）: ¥780/月
AI生成コスト/ユーザー: ~¥30/月（月10パック × 8枚 × ¥0.3）
インフラ費/ユーザー: ~¥5/月
粗利: ¥745/月 (95.5%)
Stripe手数料: 3.6% = ¥28
純利: ¥717/ユーザー/月

LTV（想定8ヶ月）: ¥5,736
CAC目標: <¥1,500
LTV/CAC: >3.8x ✅

損益分岐: Creator 1人で黒字（インフラ~$4.50/月、1人の粗利~$5）
```
