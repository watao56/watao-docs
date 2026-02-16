# ⚡ SpeedLoss — ページ速度低下アラート

## 概要
Webサイトの表示速度が悪化したら即座にアラートを送る。ページ速度はSEO順位・コンバージョン率に直結するが、プラグイン追加・画像変更・サーバー劣化等で徐々に遅くなり、気づいた時にはSEO順位が落ちている。SpeedLossは定期的にLighthouse/Core Web Vitalsを計測し、閾値を超えたらアラートを送る。

## ターゲットユーザー
- **SEO担当者**: 速度低下によるランキング下落を防ぎたい
- **Webサイトオーナー**: 表示速度がコンバージョンに影響
- **WordPressサイト運営者**: プラグイン追加で知らぬ間に遅くなる
- **EC事業者**: 1秒の遅延で7%のCV低下（Amazon調査）

## 料金プラン
| プラン | 月額 | 内容 |
|--------|------|------|
| Free | $0 | 1ページ、週1回計測、メール通知 |
| Pro | $5/月 | 10ページ、日次計測、Slack通知、トレンドグラフ |
| Business | $12/月 | 50ページ、6時間ごと計測、Webhook、PDF週次レポート |

## ユーザーフロー
1. メールアドレスでサインアップ
2. 監視したいページURLを入力
3. 初回計測→ベースラインスコアを記録
4. 定期計測でCore Web Vitals (LCP, FID, CLS) + パフォーマンススコアを取得
5. ベースラインから20%以上悪化→アラート通知
6. ダッシュボードでスコア推移グラフを確認
7. 悪化原因のヒント表示（「画像サイズが増加」「JSバンドルが肥大化」等）

## アーキテクチャ
```
[ユーザー] → [Next.js on Vercel]
                    ↓
            [Supabase (認証/DB)]
                    ↓
    [EventBridge] → [Lambda (Lighthouse CI)]
                    ↓
        [PageSpeed Insights API (無料)]
                    ↓
        [スコア記録・差分検知] → [SNS → 通知]
                    ↓
        [ダッシュボード (トレンドグラフ)]
```

## DB設計
### usersテーブル
| カラム | 型 | 説明 |
|--------|-----|------|
| id | uuid | PK |
| email | varchar | メール |
| plan | enum | free/pro/business |

### pagesテーブル
| カラム | 型 | 説明 |
|--------|-----|------|
| id | uuid | PK |
| user_id | uuid | FK |
| url | varchar | ページURL |
| baseline_score | int | ベースラインスコア |
| threshold_pct | int | アラート閾値（%低下） |
| check_interval | int | 計測間隔（時間） |

### measurementsテーブル
| カラム | 型 | 説明 |
|--------|-----|------|
| id | uuid | PK |
| page_id | uuid | FK |
| measured_at | timestamp | 計測日時 |
| performance_score | int | パフォーマンススコア |
| lcp_ms | int | Largest Contentful Paint |
| fid_ms | int | First Input Delay |
| cls | float | Cumulative Layout Shift |
| total_size_kb | int | 総ページサイズ |

## コスト見積もり
| 項目 | 月額 |
|------|------|
| Vercel (Hobby) | $0 |
| Supabase (Free) | $0 |
| Lambda | ~$0.50 |
| PageSpeed Insights API | $0（無料、25,000回/日） |
| EventBridge | ~$0.01 |
| SES | ~$0.10 |
| **合計** | **~$0.61** |

AIコスト: なし（AI不使用）

## MVPスコープ
- メール認証サインアップ
- ページURL登録
- PageSpeed Insights APIで週次計測
- パフォーマンススコア + Core Web Vitals記録
- ベースラインからの悪化検知→メール通知
- シンプルなスコア履歴表示

## マーケティング計画
- **SEO**: 「サイト速度 低下 原因」「Core Web Vitals 改善」「WordPress 遅くなった」
- **Twitter/X**: SEO担当者・WordPress界隈に訴求
- **Zenn/Qiita**: 「Core Web Vitalsを自動監視する方法」技術記事
- **Product Hunt**: ローンチ
- **WordPress関連ブログ**: 「プラグイン入れたら速度落ちた」系の記事からリンク

## 技術スタック
- **フロント**: Next.js (App Router) + Tailwind CSS + Chart.js
- **ホスティング**: Vercel (Hobby)
- **認証/DB**: Supabase
- **計測**: Google PageSpeed Insights API (無料)
- **スケジューラ**: Amazon EventBridge Scheduler
- **バッチ**: AWS Lambda
- **通知**: Amazon SES + Slack Webhook

## リスクと対策
| リスク | 対策 |
|--------|------|
| PageSpeed API のレート制限 | 25,000回/日の無料枠は十分。超過時はキューイングで分散 |
| 計測値のブレ | 3回計測の中央値を採用。ブレの注意書きを表示 |
| 競合が多い（SpeedCurve等） | 高機能・高価格帯。「アラートだけ」に特化して$5 |
| ユーザーが改善方法を知らない | 基本的な改善ヒントを自動表示（画像圧縮、JS削減等） |

## 競合分析
| サービス | 違い |
|----------|------|
| SpeedCurve | 月$20+。高機能だが高い |
| GTmetrix Pro | 月$15+。同上 |
| PageSpeed Insights（手動） | 手動で毎回確認。アラートなし |
| DebugBear | 月$12+。高い |

**差別化**: 「速度悪化のアラート」だけに特化。月$5。自動で監視し続けてくれる安心感。

## $20達成シナリオ
- **Proプラン($5) × 4人 = $20/月**
- SEO担当者やWordPressサイトオーナーがメインページの速度を監視
- SEO順位低下の損失 >> 月$5
- Free→Pro転換率15%で、Free 27人 → Pro 4人
- SEO記事 + Qiita/Zenn記事で月25サインアップ → 3ヶ月で達成

## ユニットエコノミクス
| 指標 | 値 |
|------|-----|
| ARPU | $5/月 |
| 月次チャーン率 | 6%（速度が安定すると解約リスク） |
| LTV | $5 ÷ 0.06 = $83 |
| CAC | $0（オーガニック集客） |
| 粗利率 | ($5 - $0.15) ÷ $5 = **97.0%** |
| LTV/CAC | ∞ |

※1ユーザーあたりインフラコスト: Lambda(10ページ×30日) + SES = ~$0.15/月
