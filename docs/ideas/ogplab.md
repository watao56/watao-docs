# 🖼️ OGP Lab — ダイナミックOGP画像生成API

## 概要

URLパラメータだけで美しいOGP画像（SNSシェア時のサムネイル）を動的生成するAPI/SaaSサービス。海外では**Cloudinary OG Image**、**imgix**、**Vercel OG**（@vercel/og）、**Bannerbear**（$40K+ MRR）が急成長中。日本語フォント・縦書き・和テイスト対応で、ブログ・技術記事・ECサイトのOGP問題を一発解決。

## 海外事例分析

### Bannerbear（bannerbear.com）
- **MRR**: $40K+（創業者Jon Yongfookが公開）
- **ユーザー数**: 数千社
- **料金**: $49-199/月
- **特徴**: テンプレートベースの画像自動生成API

### Cloudinary / imgix
- **市場規模**: 画像処理SaaS全体で$1B+
- **OG機能**: URL変換で動的画像生成

### @vercel/og (Satori)
- **GitHub Stars**: 10K+
- **特徴**: Edge Functionでリアルタイムにog:image生成
- **制限**: 自前デプロイ必須、テンプレート管理が面倒

### なぜ海外で流行っているか
1. **SNSマーケの重要性**: OGP画像のクリック率への影響は2-3倍
2. **動的コンテンツの増加**: ブログ記事ごとに手動でOGP画像を作るのは非効率
3. **ノーコードとの相性**: Webflow/Notion等でOGP画像を自動化したい
4. **開発者ツール需要**: API一発で解決するDX（Developer Experience）

### 日本で未展開/弱い理由
1. **日本語フォント問題**: Satoriは日本語フォント対応が不完全（豆腐化）
2. **縦書き未対応**: 日本語特有のレイアウトニーズ
3. **既存サービスは英語圏向け**: テンプレートが英語デザイン前提
4. **OGP意識の低さ**: 日本のブログ/メディアはOGP画像を軽視しがち

### 参入チャンス
- **Zenn/Qiita/noteユーザー**: 技術記事のOGPを美しくしたい需要
- **Next.js開発者**: @vercel/ogの日本語問題を解決するだけで価値
- **ECサイト**: 商品ページのSNSシェア画像を自動化

## ターゲット

### メインターゲット
- **個人ブロガー/技術ブロガー**: Zenn/Qiita/Hugo/Next.js等
- **Webエンジニア**: 自作サービスのOGP画像自動化
- **マーケター**: SNSシェア時のクリック率改善

### サブターゲット
- **ECサイト運営者**: 商品OGP自動生成
- **メディア/ニュースサイト**: 記事OGP自動化

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | ¥0 | 月100枚まで、透かし付き、基本テンプレート3種 |
| Pro | ¥980/月 | 月5,000枚、透かしなし、全テンプレート、カスタムフォント |
| Business | ¥2,980/月 | 月50,000枚、API優先、カスタムテンプレートエディタ、CDN配信 |

## ユーザーフロー

```
1. LP: ライブデモ（タイトル入力→即OGP画像プレビュー）
2. サインアップ → APIキー発行
3. 使い方:
   a) URL方式: https://ogp.lab/api/v1/{template}?title=記事タイトル&author=名前
   b) テンプレートエディタ: ブラウザでドラッグ&ドロップ編集
   c) コードスニペット: Next.js/Hugo/WordPress用のコピペコード
4. OGP画像がCDN経由で配信
5. ダッシュボード: 生成数・人気テンプレート・利用状況
```

## デザインコンセプト

### ブランディング
- **ブランド名**: OGP Lab（OGPの実験室）
- **タグライン**: 「シェアされる画像、1行で。」
- **ロゴ**: フレーム（OGP枠）+フラスコ（Lab）のミニマルアイコン
- **カラー**: ダークモード基調、グラデーション（オレンジ→ピンク）

### UIトーン
- **開発者フレンドリー**: コードスニペットが映えるダークUI
- **ライブプレビュー**: パラメータ変更が即座にプレビューに反映
- **テンプレートギャラリー**: 美しいOGPテンプレートを一覧表示
- **コピペ体験**: 「このURLをmetaタグに貼るだけ」の簡潔さ

### テンプレートカテゴリ
- **テックブログ風**: コード背景+タイトル+著者アイコン
- **和モダン**: 和紙テクスチャ+縦書きタイトル
- **グラデーション**: Twitter/X映えするカラフル背景
- **ミニマル**: 白背景+大きなタイポグラフィ
- **マガジン風**: 写真背景+オーバーレイテキスト

### LP設計
- ヒーロー: テキスト入力→リアルタイムOGP画像生成デモ
- 「Before/After」: デフォルトの味気ないOGP vs OGP Lab
- コードスニペット: 3行で導入できることを視覚的に
- テンプレートギャラリー: 全テンプレートのプレビュー

## アーキテクチャ

```
[クライアント/ブラウザ]
    ↓ GET /api/v1/{template}?title=xxx&...
[Cloudflare Workers] ← CDN + Edge処理
    ↓ キャッシュミス時のみ
[画像生成エンジン]
    ├── Satori (HTML→SVG) + Resvg (SVG→PNG)
    ├── 日本語フォント埋め込み (Noto Sans JP / Noto Serif JP)
    └── テンプレートレンダリング
    ↓
[Cloudflare R2] ← 生成画像キャッシュ
    ↓
[Supabase] ← ユーザー・APIキー・利用量管理
```

## DB設計

```sql
users (
  id UUID PK,
  email TEXT UNIQUE,
  api_key TEXT UNIQUE,
  plan TEXT DEFAULT 'free',
  monthly_usage INT DEFAULT 0,
  created_at TIMESTAMPTZ
)

templates (
  id UUID PK,
  slug TEXT UNIQUE,        -- "tech-blog", "wa-modern", etc.
  name TEXT,
  category TEXT,
  html_template TEXT,      -- Satori用HTMLテンプレート
  default_params JSONB,
  is_premium BOOLEAN,
  created_at TIMESTAMPTZ
)

custom_templates (
  id UUID PK,
  user_id UUID FK → users,
  name TEXT,
  html_template TEXT,
  params_schema JSONB,
  created_at TIMESTAMPTZ
)

generations (
  id UUID PK,
  user_id UUID FK → users,
  template_id UUID,
  params JSONB,
  cached BOOLEAN,
  created_at TIMESTAMPTZ
)
```

## コスト見積もり

### 固定費
| 項目 | 月額 |
|------|------|
| Cloudflare Workers (Free) | $0 |
| Cloudflare R2 (10GB Free) | $0 |
| Supabase (Free) | $0 |
| ドメイン | $1 |
| **合計** | **$1/月** |

### 変動費（100ユーザー、月10,000枚生成時）
| 項目 | 月額 |
|------|------|
| Cloudflare Workers (10M req Free) | $0 |
| R2 Storage (キャッシュ) | $0 (10GB Free枠内) |
| **合計** | **$0/月** |

**AI費用ゼロ、外部API費ゼロ**: Satori + Resvg は全てOSSライブラリ

## MVPスコープ（2週間）

### Week 1
- [ ] Cloudflare Workers + Satori セットアップ
- [ ] 日本語フォント埋め込み（Noto Sans JP / Serif JP）
- [ ] 基本テンプレート3種作成
- [ ] URL→画像生成パイプライン
- [ ] R2キャッシュ

### Week 2
- [ ] Supabase連携（ユーザー・APIキー管理）
- [ ] テンプレートエディタ（簡易版）
- [ ] ダッシュボード（利用量表示）
- [ ] Stripe決済
- [ ] LP作成（ライブデモ付き）

## マーケティング計画

### Phase 1: ローンチ
- Zennで「Next.jsのOGP画像問題を完全解決する」技術記事
- Twitter/Xで開発者向けにデモ動画投稿
- Hugo/Astroコミュニティでの紹介

### Phase 2: グロース
- WordPress プラグイン公開
- 「OGP画像でクリック率2倍」データ記事
- Qiitaアドベントカレンダーでの露出

### Phase 3: 定着
- テンプレートマーケットプレイス（ユーザー投稿）
- Figma連携（Figmaデザイン→OGPテンプレート変換）
- Webhook連携（記事公開時に自動生成）

## 技術スタック

- **画像生成**: Satori + @resvg/resvg-js
- **Edge**: Cloudflare Workers
- **ストレージ**: Cloudflare R2
- **DB**: Supabase (PostgreSQL)
- **フロント**: Next.js 14 + TailwindCSS
- **決済**: Stripe
- **フォント**: Noto Sans JP / Noto Serif JP (Google Fonts)

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| @vercel/ogの日本語対応改善 | 高 | テンプレート・CDN・APIの利便性で差別化 |
| Cloudinary/imgixの日本参入 | 中 | 低価格+日本語特化で勝負 |
| 無料枠で十分なユーザー | 中 | 月100枚制限、透かし付きで適切に制限 |
| OGP自体の重要性低下 | 低 | SNS依存は今後も続く |

## 競合分析

| サービス | 日本語 | 動的生成 | 料金 | 弱み |
|----------|--------|---------|------|------|
| @vercel/og | △（豆腐化） | ◎ | 無料 | 自前デプロイ必須、日本語問題 |
| Bannerbear | × | ◎ | $49/月〜 | 英語のみ、高額 |
| Cloudinary | △ | ○ | $89/月〜 | OGP特化でない、高額 |
| 手動（Canva等） | ◎ | × | 無料 | 自動化不可 |
| **OGP Lab** | **◎** | **◎** | **¥980/月** | 新規参入 |

## $20/月達成シナリオ

```
目標: $20/月 = 約¥3,000/月

シナリオ: Pro ¥980/月 × 4人 = ¥3,920（≒$26/月）

Timeline:
- Week 1-2: MVP開発
- Week 3: ローンチ、Zenn技術記事でバズ狙い
- Week 4-6: 無料ユーザー100人（開発者は試しやすい）
- Week 7-8: Pro転換率4% → 4人 = ¥3,920/月 ✅

コスト: $1/月（全てゼロコスト構造）→ 粗利99%以上
```
