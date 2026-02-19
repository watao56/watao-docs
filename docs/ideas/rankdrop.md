# 📉 RankDrop — SEO検索順位低下アラート

## 概要

指定したキーワードの検索順位を毎日自動チェックし、順位が急落（3位以上低下）した場合にアラートを送るサービス。SEO順位の低下は放置するほど回復が困難になり、**1位→2位に落ちるだけでCTRが約50%減少**する。早期検知→即時対応で売上への影響を最小化する。

## ターゲット

- **メイン**: アフィリエイター・ブロガー（収益がSEO順位に直結）
- **サブ**: 中小企業のWeb担当者（集客がSEO依存）
- **拡張**: Web制作会社（クライアントサイトの順位レポート）

## 料金

| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 5キーワード、週1回チェック、メール通知 |
| Pro | $5/月 | 50キーワード、毎日チェック、Slack/LINE通知、順位推移グラフ |
| Agency | $15/月 | 200キーワード、複数サイト、クライアントレポート生成 |

## ユーザーフロー

1. **登録**: メールで登録
2. **サイト追加**: 監視対象のURLを入力
3. **キーワード設定**: 追跡するキーワードを入力（「フリーランス 確定申告」等）
4. **初回チェック**: 現在の順位を取得してベースライン設定
5. **日次監視**: 毎日自動で順位チェック
6. **アラート**: 順位が閾値以上低下→通知（「"確定申告 フリーランス" が3位→8位に低下」）
7. **ダッシュボード**: 順位推移グラフ、アラート履歴

## アーキテクチャ

```
[EventBridge: 毎日6:00 AM]
         ↓
[Lambda: Rank Checker]
         ↓
[SerpApi / Google Search API]
         ↓
[DynamoDB: 順位データ保存]
         ↓
判定: 前日比 -3以上?
         ↓ Yes
[通知: SES / Slack / LINE]
         ↓
[Next.js Dashboard: 順位推移表示]
```

### コンポーネント

- **Frontend**: Next.js（Vercel無料枠）
- **API**: AWS Lambda + API Gateway
- **DB**: DynamoDB
- **順位取得**: SerpApi（$50/5000検索）またはGoogle Custom Search API
- **スケジューラ**: EventBridge（1日1回）
- **通知**: SES + Slack Webhook + LINE Notify
- **認証**: NextAuth.js

## DB設計

### Users テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | UUID |
| email | String | メール |
| plan | String | free/pro/agency |

### Sites テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| userId (PK) | String | ユーザーID |
| siteId (SK) | String | サイトID |
| url | String | 対象URL |
| name | String | サイト名 |

### Keywords テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| siteId (PK) | String | サイトID |
| keywordId (SK) | String | キーワードID |
| keyword | String | 検索キーワード |
| currentRank | Number | 現在の順位（null=圏外） |
| previousRank | Number | 前日の順位 |
| bestRank | Number | 過去最高順位 |
| alertThreshold | Number | アラート閾値（デフォルト: -3） |
| lastChecked | Number | 最終チェック時刻 |

### RankHistory テーブル
| フィールド | 型 | 説明 |
|-----------|------|------|
| keywordId (PK) | String | キーワードID |
| date (SK) | String | 日付(YYYY-MM-DD) |
| rank | Number | 順位 |
| url | String | ランクインしたURL |

## コスト見積もり

### インフラコスト（月額・初期）

| 項目 | 月額 |
|------|------|
| Vercel | $0 |
| Lambda | $0（Free Tier内） |
| DynamoDB | $0（Free Tier内） |
| API Gateway | $0.10 |
| SES | $0 |
| **インフラ合計** | **$0.10/月** |

### 外部APIコスト

| 項目 | 単価 | 月間想定 | 月額 |
|------|------|---------|------|
| SerpApi | $50/5000検索 | 初期: 100検索 | $1.00 |
| **API合計** | | | **$1.00/月** |

### 合計: $1.10/月（初期）

### 100ユーザー時（平均30キーワード/ユーザー）
| 項目 | 月額 |
|------|------|
| SerpApi | $18.00（3000キーワード × 30日 / 5000） |
| Lambda | $0.50 |
| DynamoDB | $0.50 |
| **合計** | **$19.00/月** |

**⚠️ 注意**: 100ユーザー時のSerpApi費用が高い。対策:
- Google Custom Search API（$5/1000クエリ、1日100クエリ無料）に切替
- 自前スクレイピング（リスクあり）
- Pro以上のみ毎日チェック、Freeは週1回

### 最適化後の100ユーザー時コスト
- Google Custom Search APIの無料枠活用 + SerpApiを有料ユーザーのみ
- **$8.00/月**（推定）

## MVPスコープ

### Phase 1（2週間）
- ユーザー登録/ログイン
- サイト＋キーワード登録
- SerpApiで日次順位チェック
- 順位低下アラート（メール）
- シンプルダッシュボード（キーワード一覧、現在順位）

### Phase 2（+1週間）
- 順位推移グラフ（Recharts）
- Slack / LINE通知
- Stripe決済
- キーワード提案（Google Search Console連携）

### Phase 3（+2週間）
- クライアントレポートPDF生成
- 競合サイトの順位も同時追跡
- モバイル/デスクトップ別順位

## マーケ計画

### 初期（1-3ヶ月目）
- **SEO**: 「検索順位 下がった 原因」「SEO順位 監視 無料」（meta的にSEOツールのSEO）
- **ブロガーコミュニティ**: はてなブログ、note、WordPress界隈
- **Twitter/X**: SEO系インフルエンサーへの紹介
- **Product Hunt**: 「Simple rank tracking for indie creators」

### 中期（3-6ヶ月目）
- **アフィリエイター向けLP**: 「順位が3つ落ちたら月収○万円減」の訴求
- **Web制作会社向け**: 「クライアントに月次レポートを自動送信」
- **比較記事**: 「RankTracker vs Ahrefs vs RankDrop（個人向けならこれ）」

## 技術スタック

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 14, TailwindCSS, shadcn/ui, Recharts |
| Backend | AWS Lambda (Node.js 20) |
| DB | DynamoDB |
| Auth | NextAuth.js |
| Rank API | SerpApi / Google Custom Search API |
| Hosting | Vercel + AWS |
| CI/CD | GitHub Actions |
| 決済 | Stripe |
| 通知 | SES, Slack Webhook, LINE Notify |

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| SerpApi費用がスケールしない | 粗利率低下 | 自前検索結果パーサー or Google CSE無料枠活用 |
| Google検索結果のパーソナライゼーション | 順位のブレ | 地域・言語を固定。複数回チェックで平均化 |
| 大手競合（Ahrefs, SEMrush） | 差別化困難 | 価格1/20。「個人・小規模向け」に特化 |
| Google API利用規約 | スクレイピング不可 | 正規API（SerpApi, CSE）のみ使用 |

## 競合分析

| 競合 | 月額 | 特徴 | RankDropの優位性 |
|------|------|------|-----------------|
| Ahrefs | $99〜 | 総合SEOツール | 10倍以上高い。順位監視だけなら過剰 |
| SEMrush | $129〜 | 総合SEOツール | 同上 |
| Rank Tracker (SE Ranking) | $44〜 | 順位追跡特化 | まだ高い。個人には不要な機能多い |
| GRC | 買い切り¥4,860 | PC常駐型 | PCつけっぱなし不要。モバイル対応。通知あり |
| SERPWatcher | $29〜 | 順位追跡 | 個人には高い |

## $20達成シナリオ

| シナリオ | Pro ($5) | Agency ($15) | MRR |
|---------|---------|--------------|-----|
| 最速 | 4人 | 0人 | $20 |
| 現実的（3ヶ月目） | 3人 | 1人 | $30 |
| 保守的（6ヶ月目） | 4人 | 1人 | $35 |

### 達成根拠
- SEOで稼ぐブロガー/アフィリエイターにとって順位=収入
- 既存ツールが$30-130/月の中、$5/月は圧倒的に安い
- 「順位が落ちたことに気づかず月3万円損した」は珍しくない
- 日本のブロガー/アフィリエイターは100万人以上

## ユニットエコノミクス

| 指標 | 値 |
|------|-----|
| CAC | $0-3（SEO+コミュニティ） |
| ARPU | $7.50（Pro:Agency = 3:1想定） |
| 粗利率 | 89.3%（$8.00/$75.00 @100ユーザー） |
| LTV（12ヶ月） | $90.00 |
| LTV/CAC | $30-∞ |
| 月間チャーン率 | 5%（SEO依存ビジネスなら継続的に必要） |
| 損益分岐ユーザー数 | 1人（$5 > $1.10） |

### ⚠️ スケール時の注意
100ユーザー超でSerpApiコストが急増するため、早期にGoogle CSE無料枠最大化 or 自前パーサーの検討が必要。粗利率を90%以上に維持するには、APIコストの最適化がキー。
