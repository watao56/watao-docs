# ⭐ ReviewPing — Googleレビュー新着通知＋返信リマインダー

## 概要
Googleマップのレビューが投稿されたら即座に通知し、未返信レビューをリマインドする。悪いレビューを放置すると来店率が下がり、良いレビューに返信しないと評価が伸びない。Google Business Profile APIを公式利用し、規約リスクなく安定運用。

## ターゲットユーザー
- **飲食店オーナー**: 悪レビュー放置で来店減を防ぎたい
- **美容院/サロン**: レビュー評価が集客に直結
- **クリニック/歯医者**: 口コミが患者獲得の生命線
- **Web制作会社**: クライアント店舗のレビュー管理代行

## 料金プラン
| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 1店舗、1日1回チェック、メール通知のみ |
| Pro | $5/月 | 3店舗、1時間ごとチェック、LINE/Slack通知、未返信リマインド |
| Business | $12/月 | 10店舗、15分ごと、評価トレンドレポート、低評価アラート |

## ユーザーフロー
1. Googleアカウントでサインアップ（OAuth）
2. Google Business Profile連携→店舗選択（API経由で自動取得）
3. ReviewPingがGoogle Business Profile APIで定期的にレビュー取得
4. 新着レビュー検知→即座にメール/LINE/Slack通知（星数、内容、投稿者名）
5. 24時間未返信のレビュー→リマインド通知
6. ダッシュボードで評価推移・返信率を確認
7. 低評価（★1-2）は赤アラートで緊急通知

## アーキテクチャ
```
[ユーザー] → [Next.js on Vercel]
                    ↓
        [Google OAuth → Business Profile API連携]
                    ↓
            [Supabase (認証/DB)]
                    ↓
    [EventBridge] → [Lambda (API呼び出し)]
                    ↓
        [Google Business Profile API (レビュー取得)]
                    ↓
        [新着差分検知] → [SNS → メール/LINE/Slack通知]
                    ↓
        [未返信チェック] → [リマインド通知]
```

## DB設計
### usersテーブル
| カラム | 型 | 説明 |
|--------|-----|------|
| id | uuid | PK |
| email | varchar | メール |
| plan | enum | free/pro/business |
| google_token | text | Google OAuth トークン（暗号化） |
| line_webhook | varchar | LINE通知用 |

### placesテーブル
| カラム | 型 | 説明 |
|--------|-----|------|
| id | uuid | PK |
| user_id | uuid | FK |
| google_account_id | varchar | GBP Account ID |
| google_location_id | varchar | GBP Location ID |
| name | varchar | 店舗名 |
| check_interval | int | チェック間隔（分） |

### reviewsテーブル
| カラム | 型 | 説明 |
|--------|-----|------|
| id | uuid | PK |
| place_id | uuid | FK |
| google_review_id | varchar | Google側のID |
| author | varchar | 投稿者名 |
| rating | int | 星数 |
| text | text | レビュー本文 |
| posted_at | timestamp | 投稿日時 |
| has_reply | boolean | 返信済みか |
| notified_at | timestamp | 通知送信日時 |

## コスト見積もり
| 項目 | 月額 |
|------|------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| Lambda | ~$0.30 |
| Google Business Profile API | $0（無料） |
| SES | ~$0.10 |
| EventBridge | ~$0.01 |
| **合計** | **~$0.41** |

AIコスト: なし（AI不使用）

## MVPスコープ
- Google OAuth サインアップ
- Google Business Profile API連携→店舗自動取得
- 1日1回のレビュー差分チェック
- 新着レビューのメール通知（星数・本文・投稿者）
- 低評価（★1-2）の優先アラート
- シンプルなダッシュボード（最新レビュー一覧）

## マーケティング計画
- **SEO**: 「Googleレビュー 通知」「口コミ 管理 ツール」「悪いレビュー 対処法」
- **Twitter/X**: 飲食店経営者向け「レビュー返信で評価が上がるデータ」の情報発信
- **Instagram**: 店舗オーナー向けインフォグラフィック
- **Web制作会社パートナー**: クライアント店舗のレビュー管理代行ツールとして提案（BtoB）
- **Google Business Profile関連YouTube**: 「レビュー管理の自動化」としてコラボ
- **飲食店向けメディア（食べログ系ブログ）**: 寄稿

## 技術スタック
- **フロント**: Next.js (App Router) + Tailwind CSS
- **ホスティング**: Vercel (Hobby)
- **認証/DB**: Supabase (Google OAuth)
- **レビュー取得**: Google Business Profile API（公式）
- **スケジューラ**: Amazon EventBridge Scheduler
- **バッチ**: AWS Lambda
- **通知**: Amazon SES + LINE Messaging API + Slack Webhook

## リスクと対策
| リスク | 対策 |
|--------|------|
| Google Business Profile APIの利用制限 | 公式APIなので安定。レート制限は十分な枠がある |
| Google OAuth審査 | 審査に時間がかかる場合あり。早期に申請を開始 |
| 店舗オーナーへのリーチ | **Web制作会社をパートナーチャネルに**。BtoBで安定的に獲得 |
| GBP通知機能の強化による競合 | 「未返信リマインド」「トレンドレポート」「LINE通知」等の付加価値で差別化 |

## 競合分析
| サービス | 違い |
|----------|------|
| 口コミコム | 月額数万円。大企業向け。機能過多 |
| Googleビジネスプロフィール通知 | 通知が不安定。LINE非対応。リマインド機能なし |
| ReviewTrackers | 英語圏。月$50+。日本対応弱い |

**差別化**: 公式API利用で安定。「通知+未返信リマインド」に特化。月$5。Web制作会社経由のBtoBチャネル。

## $20達成シナリオ
- **Proプラン($5) × 4人 = $20/月**
- Web制作会社がクライアント店舗管理ツールとして導入（BtoBチャネル）
- 飲食店オーナーが直接利用（BtoCチャネル）
- Free→Pro転換率15%で、Free 27人 → Pro 4人
- SEO + Web制作会社パートナー で月20サインアップ → 3ヶ月で達成

## ユニットエコノミクス
| 指標 | 値 |
|------|-----|
| ARPU | $5/月 |
| 月次チャーン率 | 5% |
| LTV | $5 ÷ 0.05 = $100 |
| CAC | $0（オーガニック集客 + パートナーチャネル） |
| 粗利率 | ($5 - $0.10) ÷ $5 = **98.0%** |
| LTV/CAC | ∞ |

※1ユーザーあたりインフラコスト: Lambda(3店舗×24回/日×30日) + SES = ~$0.10/月（公式API利用でスクレイピングコスト削減）
