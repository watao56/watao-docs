# 🔄 RetroFlow — AIファシリテーション付きチーム振り返りボード

## 概要

スプリント振り返り（レトロスペクティブ）をオンラインで楽しく効率的に行うSaaS。付箋型ボードにAIファシリテーターが参加し、議論の要約・アクションアイテム抽出・感情分析・改善トレンド可視化を自動で行う。リモートチームの「振り返りがマンネリ化」「ファシリが下手」問題を解決。

## 海外事例分析

| サービス | 規模 | 特徴 |
|---------|------|------|
| **Retrium** | $5M+ ARR | レトロ特化SaaS。企業向け$29/人/月 |
| **EasyRetro (旧FunRetro)** | $2M+ ARR | シンプルなレトロボード。フリーミアム |
| **Parabol** | $3M+ ARR | レトロ+スタンドアップ。$6/人/月 |
| **Metro Retro** | $1M+ ARR | ビジュアル重視のレトロボード |

**日本市場の機会**: 日本のエンジニアチームはMiro/付箋で手動レトロが主流。日本語に完全対応したレトロ特化ツールは存在しない。「AIが空気を読んで議論をファシリする」は日本のチーム文化にフィット（遠慮して本音が出にくい問題を解決）。

## ターゲット

### プライマリ
- **スクラムチーム（3-10人）**: 定期的なスプリントレトロが必須
- **リモート/ハイブリッドチーム**: オンライン振り返りの質を上げたい
- **スクラムマスター/EM**: ファシリテーションを楽にしたい

### セカンダリ
- ワークショップ講師
- プロジェクトマネージャー
- 教育現場（授業振り返り）

## 料金

| プラン | 月額 | 内容 |
|-------|------|------|
| **Free** | ¥0 | 月3回レトロ、5人まで、基本テンプレート3種 |
| **Team** | ¥1,480/月 | 無制限レトロ、15人まで、AIファシリ、全テンプレート、トレンド分析 |
| **Business** | ¥3,980/月 | 無制限人数、Slack/Jira連携、エクスポート、SSO |

## ユーザーフロー

```
1. サインアップ（Google OAuth）
2. チーム作成 → メンバー招待（URLリンク）
3. レトロ作成 → テンプレート選択
   - KPT（Keep/Problem/Try）
   - 4Ls（Liked/Learned/Lacked/Longed for）
   - Start/Stop/Continue
   - Mad/Sad/Glad
   - カスタム
4. 匿名で付箋投稿（タイマー付き）
5. AIが類似付箋をグルーピング提案
6. 投票（ドット投票）で優先度決定
7. AIが議論サマリー＆アクションアイテム自動抽出
8. アクションアイテムをJira/Notionに連携
9. 過去レトロとの比較トレンドダッシュボード
```

## デザインコンセプト

- **"楽しいレトロ体験"**: ポップなカラー、付箋がふわっと表示されるアニメーション
- リアルタイムコラボ（カーソル表示、同時編集）
- 匿名性の視覚的担保（アバターがランダム動物アイコン）
- AIファシリテーターが吹き出しでコメント
- 感情ヒートマップ（チームの気分を色で可視化）
- コンフェッティ演出（レトロ完了時）

## アーキテクチャ

```
[Webアプリ (Next.js)] ←→ [WebSocket (Supabase Realtime)]
         ↓
   [API (Next.js API Routes)]
         ↓
   [Supabase (PostgreSQL + Auth)]
         ↓
   [AI処理 (OpenAI GPT-4o-mini)]
     - グルーピング提案
     - サマリー生成
     - アクションアイテム抽出
     - 感情分析
```

## DB設計

```sql
teams (
  id UUID PK,
  name TEXT,
  owner_id UUID FK → users,
  plan TEXT DEFAULT 'free',
  stripe_customer_id TEXT,
  created_at TIMESTAMPTZ
)

team_members (
  team_id UUID FK → teams,
  user_id UUID FK → users,
  role TEXT DEFAULT 'member', -- owner/admin/member
  PRIMARY KEY (team_id, user_id)
)

retros (
  id UUID PK,
  team_id UUID FK → teams,
  title TEXT,
  template TEXT, -- kpt/4ls/ssc/msg/custom
  status TEXT DEFAULT 'collecting', -- collecting/grouping/voting/discussing/done
  timer_seconds INT,
  ai_summary TEXT,
  ai_action_items JSONB,
  sentiment_score FLOAT, -- -1.0 to 1.0
  created_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ
)

cards (
  id UUID PK,
  retro_id UUID FK → retros,
  column_type TEXT, -- keep/problem/try etc
  content TEXT,
  author_id UUID FK → users, -- 内部参照のみ（匿名表示）
  group_id UUID, -- グルーピング用
  votes INT DEFAULT 0,
  sentiment TEXT, -- positive/neutral/negative
  created_at TIMESTAMPTZ
)

action_items (
  id UUID PK,
  retro_id UUID FK → retros,
  content TEXT,
  assignee_id UUID FK → users,
  status TEXT DEFAULT 'open', -- open/in_progress/done
  due_date DATE,
  external_id TEXT, -- Jira/Notion連携用
  created_at TIMESTAMPTZ
)
```

## コスト見積もり

| 項目 | 月額（50ユーザー時） |
|------|---------------------|
| Vercel (Hobby) | $0 |
| Supabase (Free → Pro) | $0-25 |
| OpenAI API (GPT-4o-mini) | ~$2（月100レトロ × $0.02/レトロ） |
| ドメイン | ~$1 |
| **合計** | **~$3-28/月** |

※初期はSupabase Free tier ($0)で運用。Realtime接続数200超えたらPro ($25)へ。

## MVPスコープ（2週間）

### Week 1
- [ ] Auth（Google OAuth）
- [ ] チーム作成＆招待
- [ ] KPTテンプレートでレトロ作成
- [ ] リアルタイム付箋投稿（Supabase Realtime）
- [ ] ドット投票

### Week 2
- [ ] AIサマリー＆アクションアイテム抽出
- [ ] 匿名モード
- [ ] タイマー機能
- [ ] Stripe決済
- [ ] LP

## マーケ計画

### フェーズ1（ローンチ〜1ヶ月）
- Qiita/Zenn にスクラムレトロ改善記事
- Twitter/X でスクラムマスター/EM向け発信
- アジャイルコミュニティ（Regional Scrum Gathering等）でPR

### フェーズ2（2-3ヶ月）
- SEO:「振り返り ツール」「レトロスペクティブ オンライン」
- Slack App Directory登録
- スクラム研修講師へのアプローチ

### フェーズ3（3-6ヶ月）
- Jira/Linear/Notion連携
- エンタープライズ機能（SSO、監査ログ）
- 英語版展開

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| フレームワーク | Next.js 14 |
| リアルタイム | Supabase Realtime (WebSocket) |
| スタイリング | Tailwind CSS + shadcn/ui + Framer Motion |
| AI | OpenAI GPT-4o-mini |
| DB/Auth | Supabase |
| 決済 | Stripe |
| デプロイ | Vercel |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| Miro/Notionで十分と思われる | 採用されない | レトロ特化のUXで差別化 |
| リアルタイム同期の複雑さ | バグ | Supabase Realtimeで簡素化 |
| チーム単位課金でCAC高い | 成長遅い | 個人Free→チーム導入の導線 |
| AI日本語精度 | 低品質サマリー | GPT-4o-miniの日本語能力は十分 |

## 競合分析

| 競合 | 強み | 弱み | RetroFlowの優位性 |
|------|------|------|-------------------|
| Retrium | 機能豊富 | $29/人/月と高額、英語のみ | 日本語対応、¥1,480/チーム |
| EasyRetro | シンプル | AI機能なし、英語 | AIファシリ、日本語 |
| Miro | 汎用ボード | レトロ特化でない | レトロに最適化されたUX |
| 手動(Mural+付箋) | 自由度 | 準備・まとめが手間 | 自動サマリー＆アクション抽出 |

## $20達成シナリオ

```
目標: $20/月 ≒ ¥3,000/月

シナリオ: Team 2チーム
- Team ¥1,480 × 2 = ¥2,960 ≒ $20
- 必要Free登録: ~30チーム（Team転換率7%）
- 達成時期: ローンチ後2-3ヶ月

楽観シナリオ:
- Team 3 + Business 1 = ¥8,420 ≒ $56
- ローンチ後2ヶ月
```

## ユニットエコノミクス

```
ARPU（Team）: ¥1,480/月
AI費/チーム: ~¥30/月（月4レトロ × ¥8）
インフラ費/チーム: ~¥10/月
粗利: ¥1,440/月 (97.3%)
Stripe手数料: 3.6% = ¥53
純利: ¥1,387/チーム/月

LTV（想定12ヶ月）: ¥16,644
CAC目標: <¥3,000
LTV/CAC: >5.5x ✅

損益分岐: Team 1チームで黒字（Supabase Free時）
```
