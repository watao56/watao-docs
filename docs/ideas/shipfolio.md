# 🚀 ShipFolio — ビルダー向け「作ったもの」ポートフォリオ

## 概要

エンジニア・デザイナー・メイカーが「自分が作ったプロダクト」を美しく展示するポートフォリオサービス。海外の**Indie Hackers プロフィール**や**Peerlist**、**Read.cv**で急成長中の「ビルダー・アイデンティティ」文化を日本に持ち込む。GitHub/Figma/デプロイURLを繋ぐだけで自動的に美しいポートフォリオが生成される。

## 海外事例分析

### Peerlist（peerlist.io）
- **ユーザー数**: 50万人+
- **資金調達**: $2.1M
- **特徴**: プロフェッショナル向けソーシャルプロフィール、プロジェクト展示
- **料金**: 無料 + Pro $5/月

### Read.cv（read.cv）
- **特徴**: 美しいミニマルデザインのプロフィール
- **ユーザー数**: 10万人+
- **料金**: 無料

### Bento.me
- **特徴**: リンクインバイオのビジュアル版、グリッドレイアウト
- **MRR**: $50K+推定
- **料金**: 無料 + Pro $3/月

### なぜ海外で流行っているか
1. **「ビルダー」文化**: 海外IT業界で「何を作ったか」が最重要評価基準に
2. **LinkedInの限界**: テキスト職歴では伝わらない「作品」を見せたい
3. **ソーシャルプルーフ**: Product Huntアップボート、GitHub Stars等を一覧
4. **採用市場の変化**: ポートフォリオ > 履歴書の流れ
5. **デザインの民主化**: 美しいプロフィールが誰でも作れる

### 日本で未展開/弱い理由
1. **Wantedlyの支配**: 日本のIT転職＝Wantedly、ポートフォリオ文化が弱い
2. **「作ったもの」を見せる習慣がない**: 日本の職務経歴書文化
3. **Peerlist/Read.cvは英語のみ**: 日本語プロフィールが作れない
4. **デザインテンプレートの不足**: 日本向けの美しいポートフォリオサービスがない

### 参入チャンス
- **副業/フリーランス解禁**: 自分のスキルを見せる場の需要急増
- **個人開発ブーム**: 日本でもIndie Hackerコミュニティが成長中
- **転職市場のポートフォリオ化**: 「GitHubを見せて」が面接の常識に

## ターゲット

### メインターゲット
- **エンジニア**（25-35歳）: 転職・副業獲得のためのポートフォリオ
- **個人開発者**: 作ったプロダクトの実績展示
- **デザイナー**: Figma/Dribbble連携での作品展示

### サブターゲット
- **学生**: 就活用ポートフォリオ
- **クリエイター**: YouTube/ブログ等の制作実績

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | ¥0 | 5プロジェクトまで、基本テーマ |
| Pro | ¥480/月 | 無制限プロジェクト、プレミアムテーマ、カスタムドメイン、アナリティクス |
| Team | ¥980/人/月 | チームページ、メンバー一覧、採用ページ |

## ユーザーフロー

```
1. LP: 実際のShipFolioページのデモ → 「あなたのページを作る」
2. GitHub/Googleでサインアップ
3. GitHub連携 → リポジトリ自動インポート
4. プロジェクト追加: URL + スクリーンショット自動取得 + 説明
5. プロフィール編集: 自己紹介、スキル、SNSリンク
6. テーマ選択: ミニマル / カード / ベント / マガジン
7. 公開: shipfolio.dev/{username}
8. OGP自動生成 → SNSシェア
```

## デザインコンセプト

### ブランディング
- **ブランド名**: ShipFolio（Ship + Portfolio）
- **タグライン**: 「作ったもの、全部見せよう。」
- **ロゴ**: 紙飛行機（Ship）がフォルダから飛び立つモーション
- **カラー**: ブラック+ネオンパープル、テック感のある配色

### UIトーン
- **ベントスタイル**: Bento.meのグリッドレイアウトを参考
- **カード主義**: 各プロジェクトが美しいカードとして表示
- **ホバーエフェクト**: マウスオーバーでスクリーンショットが動的に変化
- **OGP映え**: SNSシェア時に美しいOGP画像が自動生成

### LP設計
- ヒーロー: 3つの異なるスタイルのポートフォリオが回転表示
- 「30秒でポートフォリオ完成」のステップ紹介
- 実際のユーザーページギャラリー
- CTA: 「GitHubで始める」ワンクリック

## アーキテクチャ

```
[Next.js on Vercel]
    ↓
[Supabase]
    ├── PostgreSQL: ユーザー・プロジェクト
    ├── Auth: GitHub/Google OAuth
    └── Storage: スクリーンショット
    
[GitHub API]
    └── リポジトリ情報・Stars取得
    
[スクリーンショット]
    └── Puppeteer on Vercel Edge / 外部API (screenshotapi.net Free枠)
    
[OGP生成]
    └── @vercel/og (Satori)
```

## DB設計

```sql
users (
  id UUID PK,
  email TEXT UNIQUE,
  username TEXT UNIQUE,
  display_name TEXT,
  bio TEXT,
  avatar_url TEXT,
  github_username TEXT,
  twitter_handle TEXT,
  website_url TEXT,
  skills TEXT[],
  theme TEXT DEFAULT 'minimal',
  plan TEXT DEFAULT 'free',
  created_at TIMESTAMPTZ
)

projects (
  id UUID PK,
  user_id UUID FK → users,
  title TEXT,
  description TEXT,
  url TEXT,
  github_url TEXT,
  screenshot_url TEXT,
  tags TEXT[],
  github_stars INT,
  position INT,           -- 表示順
  is_featured BOOLEAN,
  created_at TIMESTAMPTZ
)

analytics (
  id UUID PK,
  user_id UUID FK → users,
  page_views INT DEFAULT 0,
  date DATE,
  referrers JSONB
)
```

## コスト見積もり

### 固定費
| 項目 | 月額 |
|------|------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| ドメイン | $1 |
| **合計** | **$1/月** |

### 変動費（200ユーザー時）
| 項目 | 月額 |
|------|------|
| スクリーンショットAPI | $0 (Free枠内) |
| Supabase Storage | $0 (Free枠内) |
| GitHub API | $0 (OAuth無料) |
| **合計** | **$0/月** |

**AI費用ゼロ、外部API費ゼロ**

## MVPスコープ（2週間）

### Week 1
- [ ] Next.js + Supabase セットアップ
- [ ] GitHub OAuth認証
- [ ] プロジェクト登録UI（URL入力→スクショ自動取得）
- [ ] プロフィール編集
- [ ] 公開ページ表示

### Week 2
- [ ] テーマ切替（3種）
- [ ] OGP自動生成（@vercel/og）
- [ ] GitHub Stars自動取得
- [ ] アナリティクス（ページビュー）
- [ ] Stripe決済・LP・デプロイ

## マーケティング計画

### Phase 1: ローンチ
- Twitter/Xで「#ShipFolio」タグ、個人開発者コミュニティに投下
- Zennで「エンジニアポートフォリオ2026」記事
- connpassのLTイベントで紹介

### Phase 2: グロース
- 「#今日の個人開発」コミュニティとの連携
- Wantedly代替としてのポジショニング記事
- デザイナー向けテーマコンペ開催

### Phase 3: 定着
- Team機能でスタートアップの採用ページ
- API連携（Notion、GitHub Actions）
- 年間ベストShipFolioアワード

## 技術スタック

- **フロントエンド**: Next.js 14 + TailwindCSS + Framer Motion
- **DB**: Supabase (PostgreSQL)
- **認証**: Supabase Auth (GitHub/Google)
- **OGP**: @vercel/og (Satori)
- **決済**: Stripe
- **ホスティング**: Vercel

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| Wantedlyがポートフォリオ機能強化 | 中 | 「プロダクト展示」特化で差別化 |
| GitHubプロフィールREADMEで十分 | 中 | ビジュアル・デザイン性で上回る |
| 無料ユーザーが転換しない | 中 | カスタムドメイン・テーマをPro限定に |
| ニッチすぎる | 低 | 日本のエンジニア人口100万人以上 |

## 競合分析

| サービス | 日本語 | プロダクト展示 | デザイン | 料金 |
|----------|--------|-------------|---------|------|
| Peerlist | × | ◎ | ◎ | $5/月 |
| Read.cv | × | ○ | ◎ | 無料 |
| Wantedly | ◎ | △ | ○ | 無料 |
| GitHub Profile | ◎ | △ | × | 無料 |
| **ShipFolio** | **◎** | **◎** | **◎** | **¥480/月** |

## $20/月達成シナリオ

```
目標: $20/月 = 約¥3,000/月

シナリオ: Pro ¥480/月 × 7人 = ¥3,360（≒$22/月）

Timeline:
- Week 1-2: MVP開発
- Week 3: ローンチ、Twitter/Zennで拡散
- Week 4-6: 無料ユーザー300人（エンジニアコミュニティ）
- Week 7-10: Pro転換率2.5% → 7-8人 = ¥3,360+/月 ✅

コスト: $1/月 → 粗利99%以上
```
