# 🌱 NiwaLog — ソーシャル・デジタルガーデン

## 概要

「ブログでもSNSでもない、自分だけの庭」。思考・メモ・学びを植物のようにビジュアルに育てて公開できるナレッジ共有プラットフォーム。海外で急成長中の**Digital Garden**ムーブメント（Obsidian Publish, Quartz, are.na等）を、技術者でなくても使えるホスティッドサービスとして日本語で提供。

## 海外事例分析

### Digital Gardenムーブメント
- **Obsidian Publish**: $8/月、数万ユーザー
- **Quartz**: Hugo系の静的サイトジェネレーター、GitHub Stars 8K+
- **Are.na**: クリエイター向けビジュアルブックマーク、$45/年
- **Notion公開ページ**: 非公式だが多くの人がデジタルガーデンとして利用
- **Maggie Appleton**: デジタルガーデンのパイオニア、概念を広めた

### なぜ海外で流行っているか
1. **SNS疲れ**: タイムライン消費への反動、「自分のペースで育てる」
2. **PKM（個人知識管理）ブーム**: Obsidian/Logseq/Roam Research等の隆盛
3. **思考の可視化欲求**: グラフビュー・相互リンクで「知のネットワーク」を見たい
4. **「完成」の呪縛からの解放**: ブログは完成品を出すプレッシャー、ガーデンは途中でOK
5. **ポートフォリオの進化**: エンジニア/デザイナーが「考え方」を見せる場

### 日本で未展開/弱い理由
1. **技術的ハードル**: Obsidian PublishやQuartzはギーク向け、一般人には難しい
2. **日本語コンテンツ少**: デジタルガーデンの概念自体の認知度が低い
3. **note.comの支配**: 日本のテキスト公開＝noteが圧倒的、ガーデン的機能はなし
4. **ビジュアル表現の差**: 日本人好みの美しい表示テーマがない

### 参入チャンス
- **noteとの差別化**: ガーデンは「時系列ではなくネットワーク」、全く異なる体験
- **Zennユーザーの発展先**: 技術記事の延長で「自分の知識体系」を見せたい需要
- **非エンジニアにも**: 技術不要のホスティッドサービスは海外にも少ない

## ターゲット

### メインターゲット
- **エンジニア/デザイナー**（25-40歳）: 学習メモ・技術知見の公開
- **ライター/ブロガー**: 「未完成でも公開」の新しい表現
- **学生/研究者**: 論文メモ・学習記録のビジュアル管理

### サブターゲット
- **読書家**: 読書メモの蓄積と公開
- **クリエイター**: インスピレーション・リファレンスの整理

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | ¥0 | 50ノードまで、基本テーマ1種 |
| Gardener | ¥580/月 | 無制限ノード、プレミアムテーマ、カスタムドメイン |
| Forest | ¥1,280/月 | Gardener全機能 + アナリティクス、パスワード保護、API |

## ユーザーフロー

```
1. LP: 美しいデモガーデンが表示 → 「あなたの庭を始める」
2. サインアップ → ガーデン名決定（例: garden.niwalog.app/watao）
3. 最初のノート作成: Markdownエディタ + ウィキリンク [[リンク]]
4. グラフビュー: ノート間の繋がりが星座のように可視化
5. テーマ選択: 和紙風 / ネオン / ミニマル / ボタニカル
6. 公開 → URL共有 → 他の人のガーデンを散歩（Explore機能）
```

## デザインコンセプト

### ブランディング
- **ブランド名**: NiwaLog（庭 + Log）
- **タグライン**: 「知を育てる、自分だけの庭。」
- **ロゴ**: 幾何学的な葉っぱ/ノードがネットワークを形成するシンボル
- **カラー**: ディープグリーン+クリーム白、植物的なアースカラー

### UIトーン
- **有機的**: 直線ではなく曲線、グリッドではなくフロー
- **呼吸するUI**: ノードがゆっくり浮遊するグラフビュー
- **和の余白**: 日本的な余白感を大切にしたタイポグラフィ
- **テーマの多様性**: 和紙風（伝統）、ネオン（サイバーパンク）、ボタニカル（自然）

### LP設計
- ヒーロー: インタラクティブなグラフビューデモ（ノードをクリックすると展開）
- コンセプト説明: 「ブログは完成品。ガーデンは生きている。」
- ギャラリー: 実際のガーデン事例（デモデータ）
- CTA: 「30秒で庭を作る」

## アーキテクチャ

```
[Next.js on Vercel]
    ↓
[Supabase]
    ├── PostgreSQL: ノート・ユーザー・グラフデータ
    ├── Auth: メール/GitHub/Google認証
    └── Storage: 画像
    
[グラフ可視化]
    └── D3.js / react-force-graph

[Markdownパーサー]
    └── remark + rehype + wikilink plugin
    
[テーマエンジン]
    └── CSS Variables + TailwindCSS
```

## DB設計

```sql
users (
  id UUID PK,
  email TEXT UNIQUE,
  username TEXT UNIQUE,    -- garden.niwalog.app/{username}
  plan TEXT DEFAULT 'free',
  garden_name TEXT,
  theme TEXT DEFAULT 'minimal',
  custom_domain TEXT,
  created_at TIMESTAMPTZ
)

notes (
  id UUID PK,
  user_id UUID FK → users,
  slug TEXT,
  title TEXT,
  content TEXT,           -- Markdown with [[wikilinks]]
  status TEXT,            -- seed/sprout/evergreen
  tags TEXT[],
  is_public BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ,
  updated_at TIMESTAMPTZ
)

links (
  id UUID PK,
  source_note_id UUID FK → notes,
  target_note_id UUID FK → notes,
  UNIQUE(source_note_id, target_note_id)
)

-- バックリンク・グラフはlinksテーブルから動的に構築
```

## コスト見積もり

### 固定費
| 項目 | 月額 |
|------|------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| ドメイン | $1 |
| **合計** | **$1/月** |

### 変動費（100ユーザー時）
| 項目 | 月額 |
|------|------|
| Supabase Storage (画像) | $0 (Free枠内) |
| Vercel帯域 | $0 (100GB Free枠内) |
| **合計** | **$0/月** |

**AI費用ゼロ**: グラフ可視化・Markdownパースは全てクライアントサイド処理

## MVPスコープ（2週間）

### Week 1
- [ ] Next.js + Supabase + TailwindCSS セットアップ
- [ ] ユーザー認証・ガーデンURL生成
- [ ] Markdownエディタ（wikilink対応）
- [ ] ノート作成・編集・一覧

### Week 2
- [ ] グラフビュー（D3.js force-directed graph）
- [ ] 公開ページ表示
- [ ] テーマ切替（3種）
- [ ] LP作成
- [ ] Stripe決済・デプロイ

## マーケティング計画

### Phase 1: ローンチ
- Zennで「デジタルガーデンのすすめ」技術記事
- Twitter/Xで「#デジタルガーデン」ハッシュタグ普及
- Obsidianコミュニティへの紹介

### Phase 2: グロース
- 「ガーデンExplore」機能で発見→登録の好循環
- note.com→NiwaLogへの移行ガイド
- エンジニア勉強会でのLT

### Phase 3: 定着
- コミュニティガーデン（チーム知識庫）
- APIでObsidianプラグイン連携
- テーママーケットプレイス

## 技術スタック

- **フロントエンド**: Next.js 14 + TailwindCSS + D3.js
- **DB**: Supabase (PostgreSQL)
- **認証**: Supabase Auth
- **ストレージ**: Supabase Storage
- **決済**: Stripe
- **ホスティング**: Vercel
- **エディタ**: CodeMirror or Milkdown

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| Obsidian Publishの日本語強化 | 中 | 「技術不要」「Explore機能」で差別化 |
| Notionの公開機能強化 | 中 | グラフビュー・ガーデン美学で差別化 |
| 「デジタルガーデン」認知度低 | 高 | 概念啓蒙コンテンツで市場を作る |
| ニッチすぎて成長鈍化 | 中 | チーム版でB2B展開 |

## 競合分析

| サービス | 日本語 | 技術不要 | グラフ | 料金 | 弱み |
|----------|--------|---------|--------|------|------|
| Obsidian Publish | ○ | × | ◎ | $8/月 | Obsidian必須、技術者向け |
| Notion公開 | ◎ | ◎ | × | $8-10/月 | グラフなし、ガーデン向けでない |
| Are.na | × | ○ | △ | $45/年 | 英語のみ、テキスト弱い |
| note.com | ◎ | ◎ | × | 無料-¥500/月 | 時系列、ネットワーク構造なし |
| **NiwaLog** | **◎** | **◎** | **◎** | **¥580/月** | 新規、概念認知が課題 |

## $20/月達成シナリオ

```
目標: $20/月 = 約¥3,000/月

シナリオ: Gardener ¥580/月 × 6人 = ¥3,480（≒$23/月）

Timeline:
- Week 1-2: MVP開発
- Week 3: ローンチ、Zenn記事でエンジニア層獲得
- Week 4-8: 無料ユーザー200人
- Week 9-12: 有料転換率3% → 6人 = ¥3,480/月 ✅

コスト: $1/月（AI費用ゼロ）→ 粗利99%以上
```
